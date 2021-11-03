---
title: 踩中一个多年前的 OPCache bug
date: 2021-11-02 17:50:35
tags: php,opcache
categories: 问题分析
---

生活总是会在不经意间给你一个暴击。

“平台挂了！！” ，运营发来消息
打开平台首页，白页！
情况紧急，排查时间未知，赶紧准备把最近的改动回滚。
询问一圈， 啥？ 这个项目没发版？！！
线上摘一个节点，硬着头皮调试吧。
一通操作,安上照妖镜，把错误全部打开
```
error_reporting(E_ALL);
ini_set('display_errors', '1');
ini_set('display_startup_errors', '1');
```
原形毕露
```
PHP Fatal error:  Cannot redeclare class Foo in /data/pkg/foo_sdk-2.20.0.207/foo_client.php on line 20
```
都说重启大法好， 试试重启 php-fpm
What？！ 好了！
一脸懵逼。。。

***
线上问题算是暂时解决， 接下来慢慢啃这疑难杂症。
两个问题：
1. 为什么会报 “redeclare class” 的错误
2. 为什么重启就好了

***
根据报错， 在文件中加入 debug_print_backtrace() 函数可以看到调用栈， 发现有两处引入 foo_sdk 的逻辑， 使用的代码分别是：
```
//代码1. 
require_once('/data/sdk/foo_sdk/foo_client.php');
//代码2. 
require_once(realpath('/data/sdk/foo_sdk/foo_client.php'));
```
foo_sdk 目录是一个软链， 每一次发版就链接到最新版本的目录上
```
[vanmo@ /data/sdk ]$ ls -l
lrwxrwxrwx  1 root       root         45 10月 20 2021 foo_sdk -> /data/pkg/foo_sdk-2.20.0.207
[vanmo@ /data/sdk ]$ cd /data/pkg/
[vanmo@ /data/pkg ]$ ls -l
drwxrwxr-x  5 root       root         45 10月 20 2021 foo_sdk-2.20.0.207
drwxrwxr-x  5 root       root         45 9月 21 2021 foo_sdk-2.20.0.206
```
从时间上可以看到， 最近 foo_sdk 进行了一次升级。代码1还在加载老版本的代码， 而代码2加载的是最新版本的代码。

> 这里需要补充一点我司项目的依赖部署知识。 公司的基础服务组提供服务相关的SDK， 一台服务器上多个项目会共用一份SDK部署，由服务的开发人员进行维护。 此处暂不论这套方式的优劣。在此场景中， 服务SDK的更新导致业务项目崩溃。这让维护业务的人员猝不及防。

至此基本就可以确定了问题复现的流程，重启Fpm -> 请求接口让相关缓存建立 -> 更新SDK版本，软链目的地调整 -> 再请求接口 -> 报错

问题集中到了  require_once , realpath 相关函数知识上。

> require_once 源码解析
```
//https://raw.githubusercontent.com/php/php-src/PHP-5.6/Zend/zend_vm_execute.h line 2952
resolved_path = zend_resolve_path(Z_STRVAL_P(inc_filename), Z_STRLEN_P(inc_filename) TSRMLS_CC); // 解析文件路径
if (resolved_path) {
    failure_retval = zend_hash_exists(&EG(included_files), resolved_path, strlen(resolved_path)+1); // 是否已经 included
} else {
    resolved_path = Z_STRVAL_P(inc_filename);
}
```
当 require_once 的是个软链时，会通过 zend_resolve_path 获取到软链指向的真实链接。 这中间有 php 自己维护的 realpath_cache， 当软链指向调整后， realpath_cache 很快就会更新。

require_once 和 realpath 本身的逻辑是没有问题的。但是该项目使用了 Opcache 扩展。

关闭opcache，报错消失


***

难道！！ opcache 有 bug？ 软链目标变化后， opcache 一直没有更新对应路径的真实地址。

其实这个问题，早在 2013年就在github有相关讨论（一脸惭愧）

https://github.com/zendtech/ZendOptimizerPlus/issues/126

总结一下相关信息就是， opcache 并没有使用 php 内置的 realpath cache， 而是自己有一套缓存逻辑。
```
// https://github.com/zendtech/ZendOptimizerPlus/blob/3b16ca512290b0416baf02962beedc703147a97a/ZendAccelerator.c#L917

/* Instead of resolving full real path name each time we need to identify file,
 * we create a key that consist from requested file name, current working
 * directory, current include_path, etc */

```
从上述注释，可以推测相关key的计算方法为
"PARENT_SCRIPT_FOLDER:SYMLINKED_PATH(require_once里填写的文件名):INCLUDE_PATH_CONTENT"

这个key对应的缓存永不失效，除非重启php-fpm

结

#### 参考文档
* [1] 《如何正确发布PHP代码》https://blog.huoding.com/2016/05/27/515
* [2] 《Cache invalidation for scripts in symlinked folders》https://github.com/zendtech/ZendOptimizerPlus/issues/126