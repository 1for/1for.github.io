---
title: POD内存问题排查
date: 2021-02-25 16:50:35
tags: golang,容器
categories: 问题分析
---

## 起
最近新上线了一个golang项目， 使用公司的k8s。运行一段时间后，发现线上几个pod的内存使用量持续上涨。 为搞清楚其原因以及影响，花了几天的时间研究了一下。

![](https://user-images.githubusercontent.com/2173159/109094313-b30f1380-7754-11eb-8894-0a8f4251d7dc.png)

## 承
本着有问题先反省自身的原则，先看看程序中有什么地方内存泄露了。

### 第一回合
golang坊间有传
> 十次内存泄露，九次来自协程

要看是否是协程引起的内存泄露， 比较好的方式是使用 golang的性能分析工具pprof。 pprof记录了程序运行过程中的CPU使用情况、内存使用情况、协程运行情况。 此处我们直接观察程序协程的运行情况， 看协程数量是否是随着时间不断增长的。

按照大彬的文章 [实战GO内存泄露](https://segmentfault.com/a/1190000019222661)<sup>[1]</sup> 一通操作 ， 发现协程数量虽有波动， 但相对比较稳定。

首回合终，未果。

### 第二回合
检查程序自身的内存占用情况

通过 pprof heap 模块查看程序中占用内存最多代码段
> 此处要注意的是， golang 程序自身的垃圾回收间隔是2分钟， 每次操作完之后等待2分钟，再去看相关的数据。

从数据看，垃圾回收后，内存数据涨幅不大， 这个从监控图中的常驻内存曲线（rss）也可以看出来
![](https://user-images.githubusercontent.com/2173159/109095129-1c435680-7756-11eb-85ea-5805634bd9af.png)

次回合终，程序中未发现问题。

## 转
爬几层楼换换脑子回来再战。

继续排查过程中看到煎鱼的一篇文章 [为什么容器内存占用居高不下，频频 OOM（续）](https://eddycjy.com/posts/why-container-memory-exceed2/)<sup>[2]</sup>，似乎找到了眉目，开始探索是否是操作系统的Cache问题。

#### 先查看POD内存占用情况
```
# free -m
              total        used        free      shared  buff/cache   available
Mem:         128412       53226       71449           0        3736        3817
Swap:             0           0           0
```
哇撒，128G的内存，显然是宿主机的信息， 不可用。 
至于容器中的free命令为什么显示了宿主机的信息， 暂略过不表，咱通过其他方式查询

#### POD 内存统计的几个维度以及说明

| 内存指标 | 计算方式 | 备注 |
| --- | --- | --- |
| container_memory_usage_bytes  | memory.usage_in_bytes = memory.stat[rss] + memory.stat[cache] + memory.kmem.usage_in_bytes |  |
| container_memory_working_set_bytes | working_set = memory.usage_in_bytes - total_inactive_file | [源码参考](https://github.com/google/cadvisor/blob/master/container/libcontainer/handler.go#L821) |
| container_memory_rss | memory.stat[rss]  |  [源码参考](https://github.com/google/cadvisor/blob/master/container/libcontainer/handler.go#L798) |
| container_memory_cache | memory.stat[cache] | [源码参考](https://github.com/google/cadvisor/blob/master/container/libcontainer/handler.go#L797) |

可以这样理解， POD的 内存usage指标包括了 rss 以及 cache部分，rss 部分之前看过曲线了，是相对稳定的， 那问题就出在 cache 部分了

#### 尝试手动清理 Cache
引用[内核文档](https://www.kernel.org/doc/Documentation/sysctl/vm.txt)<sup>[3]</sup>里面的一段
> To free pagecache:
>        echo 1 > /proc/sys/vm/drop_caches
> To free reclaimable slab objects (includes dentries and inodes):
>        echo 2 > /proc/sys/vm/drop_caches
> To free slab objects and pagecache:
>        echo 3 > /proc/sys/vm/drop_caches
> ...
> To disable them, echo 4 (bit 2) into drop_caches.

设置 drop_caches 为 1 之后， container_memory_working_set_bytes 曲线如下图。 
![](https://user-images.githubusercontent.com/2173159/109097897-37fd2b80-775b-11eb-980d-8bb7b3722804.png)

## 合
至此，我们基本捋清楚POD内存的相关指标。

结合应用本身， 这是个提供 web api 和 grpc 服务的业务应用，主要是短链接并发。 底层不断建立 socket 连接进行 write 和 read，socket在Linux 下也就是一个文件，只要读写磁盘文件，就会被Linux 内核缓存。  

#### 最后几个疑问
* 那内存持续上涨会有什么问题吗？
    在我们的场景下， 是不会有影响的。 因为应用本身占用的内存很少，也不存在任何泄露问题。 当内存使用量（rss + cache）超出限制时， 操作系统会自动回收cache部分。

* 那如果应用确实存在泄漏，并超出限制了呢？
    容器本身有 OOM Killer 机制， 会主动杀死正在运行的进程释放内存。 具体的规则可以阅读文章[容器内存：我的容器为什么被杀了？](https://time.geekbang.org/column/article/315468)<sup>[4]</sup>
    
#### 参考文档
* [1] 《实战GO内存泄露》https://segmentfault.com/a/1190000019222661
* [2] 《为什么容器内存占用居高不下，频频 OOM（续）》https://eddycjy.com/posts/why-container-memory-exceed2/
* [3]  https://www.kernel.org/doc/Documentation/sysctl/vm.txt 
* [4] 《容器内存：我的容器为什么被杀了？》https://time.geekbang.org/column/article/315468
* [5] 《容器内存分析》 https://qinng.now.sh/container-memory/