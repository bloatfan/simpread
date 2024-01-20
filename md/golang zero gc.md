> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/eastpig/p/16209853.html) ![](https://raw.githubusercontent.com/lslz627/PicGo/master/637210-20220430121000572-1955426489.png)

 1. 什么是 zero gc
===============

指的是那些在 gc 扫描期间，不需要被 runtime 扫描的对象，扫描的数据越少，gc 扫描所花的时间就越短；zero gc 并不代表创建的对象 0 开销，该开辟的内存依旧需要的，而指的是 zero gc scan

ps1：在写基础框架的地方，要强调 zero gc

ps2：在写业务代码的地方，不要过度 zero gc，甚至不需要过度带着心里负担，因为一个业务 logic 一般不会产生海量对象

2. 怎么验证 zero gc
===============

 先看下面几个图

![](https://raw.githubusercontent.com/lslz627/PicGo/master/637210-20220430142953145-461221439.png)![](https://raw.githubusercontent.com/lslz627/PicGo/master/637210-20220430143132315-456100631.png)

![](https://raw.githubusercontent.com/lslz627/PicGo/master/637210-20220430143312362-2087847401.png)![](https://raw.githubusercontent.com/lslz627/PicGo/master/637210-20220430143552198-1318452632.png)

 ![](https://raw.githubusercontent.com/lslz627/PicGo/master/637210-20220430143637154-1654048143.png)![](https://raw.githubusercontent.com/lslz627/PicGo/master/637210-20220430143759657-913424703.png)

 从上面可以看出，如果你创建的对象包含指针（string 内部记录了指向数据的指针，你也可以用自定义一些嵌套 struct 来验证，结果是一样的），那么 gc 的时候，就需要被扫描；

如果切片的类型是一个 person 类型，而不是 person 的指针类型的话，那么 person 结构体中的指针类型越多，gc 消耗的时间越长；

如果切片的类型是一个 person 的指针类型，person 结构体中的指针类型字段多少，并不过太过影响 gc 消耗的时间；

如果你创建的对象不包含指针，那么 gc 的时候就不需要被扫描；

3. 结论
=====

在写一些组件库的时候，比如 im，它内部肯定需要维护海量用户与链接之间的关系（是一个常驻的对象），那么如果能做到这个对象 zero gc 的话，那么整体性能就会上一个很大的台阶

![](https://raw.githubusercontent.com/lslz627/PicGo/master/637210-20220430151837134-1660467047.png)

 像 bigcache，引入本地缓存，那么意味着我们的实例中必然需要常驻一个内存对象，如果每次 gc 的时候都需要扫描它，那么间接就增加了 gc pause 的时间，它通过 map 来做管理

kv 做了很大的优化，还是蛮巧妙的。

key：通过 hash 算法，将 “按常理应该用 string 类型的 key”，将其做 hash 之后，得到一个数值（非指针）

value：是一个偏移量，每一个 value 会存放到一个 byte 切片（byte 非指针），每次取的时候，通过计算偏移量得到 value，再将其 decode 成我们的对象

正如它上面最后一句所说的：本来 1w 个 value，按照常理，我们存放的多多少少都要有指针类型的，这样 gc 就要扫描他们，耗时长；现在优化之后，我们只需要维护一个指针，指向这个 []byte，这样

极大的提高了收敛比，本来扫描 1w 的，现在只要扫描 1 个，是不是达到了 zero gc 的目的

https://github.com/julienschmidt/httprouter

https://stackoverflow.com/questions/28780543/golang-zero-garbage-propagation-or-efficient-use-of-memory

https://blog.gopheracademy.com/advent-2018/avoid-gc-overhead-large-heaps/

https://www.bilibili.com/video/BV1aL4y1L7Kk?spm_id_from=333.337.search-card.all.click