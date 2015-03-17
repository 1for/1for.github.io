---
layout: post
title: Linux命令tee
date: 2015-03-23
tags: command
categories: Linux
---

之前写shell脚本的时候遇到一个问题：就是将脚本的输出重定向到文件之后在屏幕上就看不到任何信息了，还需要另开一个终端查看文件。

今天找了下解决方案，发现了tee这个命令，遂记录如下：


### tee

#### 用法
{% highlight sh %}
# tee --help
用法：tee [OPTION] ... [FILE] ...
读取标准输入到文件，同时到标准输出

-a, --append                追加在文件后面，补覆盖
-i, --ignoreinterrupts      忽略终端信号
    --help                  打印帮助信息并退出
    --version               打印版本信息并退出
    
如果文件是“-”，则再拷贝一遍到标准输出
{% endhighlight %}

#### Example
{% highlight sh %}
#ls -l | tee log
total 8
-rw-r--r-- 1 root root 106 Mar 20 18:06 a.php

#cat log
total 8
-rw-r--r-- 1 root root 106 Mar 20 18:06 a.php
{% endhighlight %}

#### 注意
tee默认不会将标准错误的内容输出到文件

##### 解决方案
将错误输入重定向到标准输出

{% highlight sh %}
#ls notexist
ls: cannot access notexist: No such file or directory
#ls notexist | tee log
ls: cannot access notexist: No such file or directory
#cat log
#ls notexist 2>&1 | tee log
ls: cannot access notexist: No such file or directory 
#cat log
ls: cannot access notexist: No such file or directory
{% endhighlight %}
