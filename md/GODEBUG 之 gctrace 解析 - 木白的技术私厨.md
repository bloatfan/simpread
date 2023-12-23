> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [cbsheng.github.io](https://cbsheng.github.io/posts/godebug%E4%B9%8Bgctrace%E8%A7%A3%E6%9E%90/)

[木白的技术私厨](/)
============

*   [Categories](/categories/ "Categories")
*   [Tags](/tags/ "Tags")
*   [Archives](/archives/ "Archives")
*   [About](/about/ "About")

GODEBUG 之 gctrace 解析
====================

* * *

*   July 8, 2019
*   [Golang](/categories/golang/)  
*   [go](/tags/go/)  
*   [GODEBUG](/tags/godebug/)  

gctrace 用途主要是用于跟踪 GC 的不同阶段的耗时与 GC 前后的内存量对比。信息比较简洁，可以用于对 runtime 本身进行调试之外，还可以观察线上应用的 GC 情况。

像 Dave Cheney 就写了一个工具 [gcvis](https://github.com/davecheney/gcvis) 专门用于分析 gctrace 的输出，通过可视化的方式展示出指标的变化。

如果你只关心每次 GC 的整体 STW 耗时，而对每次 GC 的 mark 与 markTermination 这两个阶段各自的 STW 耗时，包括不同的 mark workers 执行时间耗时不感兴趣的话。那只需要采集 memstats.pauseNs 这个指标就可以。因为它记录了两次 mark 的过程中 STW 的耗时。

下面给出 gctrace 的用法和输出结果字段详细解释。

> 以下分析基于 go 1.9.3

只需要在 run 命名前加上一个环境变量。runtime 就自动分析出 gctrace=1，然后在程序运行的时候，输出相关的统计信息。

GODEBUG 变量支持 14 个参数。在 [runtime](https://golang.org/pkg/runtime/) 包的 doc 里其实都有简单介绍。在调度器初始化方法 schedinit 里，会调用一个 parsedebugvars 方法对 GODEBUG 进行初始化。

看下这块的源码：

```
GODEBUG='gctrace=1' go run main.go
```

gctrace 取值可以等于 1，或任何大于 1 的数。

下面的输出来自取值等于 1：

顺便也贴出官方写的简单介绍

```
// 这些flag可以通过在go run 命令中设置GODEBUG变量来使用。但每个flag的不同取值对应的含义并没常量标识，都是硬编码
var debug struct {
	allocfreetrace   int32
	cgocheck         int32
	efence           int32
	gccheckmark      int32
	gcpacertrace     int32
	gcshrinkstackoff int32
	gcrescanstacks   int32
	gcstoptheworld   int32
	gctrace          int32
	invalidptr       int32
	sbrk             int32
	scavenge         int32
	scheddetail      int32
	schedtrace       int32
}

var dbgvars = []dbgVar{
	{"allocfreetrace", &debug.allocfreetrace},
	{"cgocheck", &debug.cgocheck},
  // ...
}

func parsedebugvars() {
  // ...

	for p := gogetenv("GODEBUG"); p != ""; {
    // ...
	}
  // ...
}
```

在看过 runtime 里 GC 和内存管理源码后，以上面给出的输出为例，通俗介绍下这些不同部分的含义。

**gc 252：** 这是第 252 次 gc。

**@4316.062s：** 这次 gc 的 markTermination 阶段完成后，距离 runtime 启动到现在的时间。

**0%：**当目前为止，gc 的标记工作（包括两次 mark 阶段的 STW 和并发标记）所用的 CPU 时间占总 CPU 的百分比。

**0.013+2.9+0.050 ms clock：**按顺序分成三部分，0.013 表示 mark 阶段的 STW 时间（单 P 的）；2.9 表示并发标记用的时间（所有 P 的）；0.050 表示 markTermination 阶段的 STW 时间（单 P 的）。

**0.10+0.23⁄5.4⁄12+0.40 ms cpu：**按顺序分成三部分，0.10 表示整个进程在 mark 阶段 STW 停顿时间 (0.013 * 8)；0.23⁄5.4/12 有三块信息，0.23 是 mutator assists 占用的时间，5.4 是 dedicated mark workers+fractional mark worker 占用的时间，12 是 idle mark workers 占用的时间。这三块时间加起来会接近 2.9*8(P 的个数)；0.40 ms 表示整个进程在 markTermination 阶段 STW 停顿时间 (0.050 * 8)。

**16->17->8 MB：**按顺序分成三部分，16 表示开始 mark 阶段前的 heap_live 大小；17 表示开始 markTermination 阶段前的 heap_live 大小；8 表示被标记对象的大小。

**17 MB goal：**表示下一次触发 GC 的内存占用阀值是 17MB，等于 8MB * 2，向上取整。

**8 P：**本次 gc 共有多少个 P。

补充说明两点：

一、heap_live 要结合 go 的内存管理来理解。因为 go 按照不同的对象大小，会分配不同页数的 span。span 是对内存页进行管理的基本单元，每页 8k 大小。所以肯定会出现 span 中有内存是空闲着没被用上的。

不过怎么用 go 先不管，反正是把它划分给程序用了。而 heap_live 就表示所有 span 的大小。

而程序到底用了多少呢？就是在 GC 扫描对象时，扫描到的存活对象大小就是已用的大小。对应上面就是 8MB。

二、mark worker 分为三种，dedicated、fractional 和 idle。分别表示标记工作干活时的专注程度。dedicated 最专注，除非被抢占打断，否则一直干活。idle 最偷懒，干一点活就退出，控制权让给出别的 goroutine。它们是并发标记工作里的 worker。

这块输出的源码为：

*   [< Newer](/posts/%E4%B8%80%E4%BB%BD%E8%AF%A6%E7%BB%86%E6%B3%A8%E9%87%8A%E7%9A%84go-mutex%E6%BA%90%E7%A0%81/ "一份详细注释的go Mutex源码")
*   [Older >](/posts/golang%E6%A0%87%E5%87%86%E5%BA%93sync.pool%E5%8E%9F%E7%90%86%E5%8F%8A%E6%BA%90%E7%A0%81%E7%AE%80%E6%9E%90/ "golang标准库sync.Pool原理及源码简析")

© Copyright 2017-2019 木白

*   [About](/about/ "About")

Powered by [Hugo](https://gohugo.io/) and [Whiteplain](https://github.com/taikii/whiteplain)