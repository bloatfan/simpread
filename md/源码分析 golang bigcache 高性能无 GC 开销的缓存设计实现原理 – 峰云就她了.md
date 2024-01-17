> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xiaorui.cc](https://xiaorui.cc/archives/7385)

> bigcache 是 golang 编写的高性能的缓存库，其设计很巧妙，通过数据分片解决高并发下锁竞争的问题，通过把数据存到 ringbuffer 来规避 golang gc 的开销。

`bigcache` 是 golang 编写的高性能的缓存库，其设计很巧妙，通过数据分片解决高并发下锁竞争的问题，通过把数据存到 ringbuffer 来规避 golang gc 的开销。

bigcache 内部使用分片来存储数据，每个分片内又使用 hashmap 存储 key 的索引，而真正的数据通过编码后放在 ringbuffer 里。bigcache 没有使用主流的 lru 和 lfu 缓存淘汰算法，而是使用覆盖写来覆盖老数据，在 ringbuffer 已满时，先删除老数据，再尝试插入新数据。另外还通过 gc 垃圾回收期删掉过期的数据。

由于 bigcache 里数据是存在 `[]byte` 类型的 ringbuffer 里，所以传入的 value 只能 `[]byte`，不能存储其他类型。这样应用场景很是受限，毕竟业务上缓存的对象较为复杂，如果每次存取都需要序列化和反序列化，那么在一定量级下 CPU 开销会很可观。

> bigcache 的实现原理跟 freecache、fastcache 大同小异，都使用了 ringbuffer 存放数据，可以很大程度降低 GC 的开销。这里的 ringbuffer 当然可以使用有名或匿名的 mmap 来构建，俗称堆外内存，但对于 golang gc 来说，mmap 和直接申请 `[]byte` 的 gc 开销没大区别。如果使用文件 mmap 映射，当系统一直有文件读写，势必会对 page cache 进行 page 淘汰，这样基于 mmap 构建的 ringbuffer，必然会受之影响。

bigcache 高级用法
-------------

bigcache 实现原理
-------------

这里简述下 bigcache 读写数据实现原理，写操作是把数据写到 ringbuffer 里，然后在 hashmap 里记录 key 和 ringbuffer 索引的关系，读取的时候，自然是从 hashmap 里获取 key 的 ringbuffer index 索引值，然后从 ringbuffer 获取数据。

### bigcache 中数据结构及布局

![](https://raw.githubusercontent.com/lslz627/PicGo/master/202304051153942.png)

`BigCache` 数据结构.

`cacheShard` 数据分片的结构.

数据在 ringbuffer 中的编码.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/202304051206330.png)

### 取摸算法

关于取摸的计算, 想来大家可以一般使用 `x%len` 公式，但这个性能相对低效，而像 `java jdk` 和 redis 中的取模公式为 `x&(len-1)`，当 hashtable 的长度是 2 的幂的情况下，这两者是等价的。

通过汇编可以看到，`&` 的汇编为 `3mov+1and+1sub`, `%` 的汇编为 `2mov+1cdp+1idiv` 。

根据 intel asm 的文档资料，`&` 操作只需 5 个 CPU 周期，而 `%` 最少需要 20 个 CPU 周期，显而易见，如果在意性能我们应该使用前者。

### Set 写流程

`Set` 用来添加数据到指定的 shard 分片里。bigcache 预设了 1024 个分片，社区中高性能的 cache 库会实现分片的方法来解决锁竞争的问题。

`set` 用来把数据写到 shard 的 ringbuffer 里，并设置 hashmap 索引，其流程如下。

1.  获取当前的秒级别的时间戳，这里抽象了 clock 方法，只要是为了方便的后面的单元测试 ;
2.  在 hashmap 里获取 key 以前的 ringbuffer 的 index 位置信息，如果不为 0，且在 ringbuffer 又可拿到该 entry，则进行删除 ;
3.  编码待写入 ringbuffer 里的结构 ;
4.  尝试把编码的数据写到 ringbuffer 里，如果空间小于 max 值，则会扩容，当无法扩容时，写入失败，说明无空闲空间，则尝试剔除最老的数据，然后再进行写入。

`removeOldestEntry` 在过期扫描部分再分析其实现.

#### resetKeyFromEntry 把 entry 中 hashcode 置为 0.

entry 的 [8:16] 字节存储了数据的 key hashcode，通过 `resetKeyFromEntry` 方法则可以把 hashcode 置为 0.

#### 对数据进行编码

![](https://raw.githubusercontent.com/lslz627/PicGo/master/202304051322303.png)

把要写到 []byte ringbuffer 的数据进行编码。

*   第一个字段为 8 字节，表明时间戳
*   第二个字段为 8 字节，是用 fnv(key) 拿到的 hashcode
*   第三个子弹为 2 字节，表明 key length

GC 垃圾回收通过时间戳来判断是否过期，通过 iterator 遍历缓存数据时，会过滤掉 hashcode 为 0 的数据，当 delete 删除数据时，会重置 hashcode 为 0。通过 key length 拿到 key。

那么如何拿到 key 的 value ? 这是因为存到 ringbuffer 时，还会加一个总大小的 length。那么获取 value 的方法显而易见了。

#### ringbuffer 写时扩容

第一种库容的场景.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/202304051929082.png)

第二种库容的场景.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/202304051930272.png)

**ringbuffer 扩容代码**

### Get 读取流程

`Get` 用来获取数据，计算 hashcode，获取对应的 shard，然后调用 `get()` 读取。

`get` 用来从 shard 里获取数据，其流程是先从 ringbuffer 里获取编码过的数据，然后通过解码获取 value。

> 不习惯 bigcache 的命名，key 的值可以为 value，这里用来 entry 这个单词。

#### getWrappedEntry 从 ringbuffer 获取数据 wrappedEntry

#### readEntry 解码获取 key 的 value

### Delete 删除 kv 流程

`Delete` 用来删除数据，先获取 key hashcode 对应分片，再执行分片的 del 进行数据删除。

`del` 用来在分片中删除数据，其流程如下。

1.  预先使用读锁检查值是否存在，如果存在走删除流程 ;
2.  根据 key hashcode 获取 ringbuffer index ;
3.  从 ringbuffer 里获取编码过的 entry ;
4.  删除 key hashcode 对应的位置 ;
5.  把 entry 的 hash 值编码成 0。

这里先使用读锁来预检查 key 是否存在，存在后，再使用写锁来数据删除。首先 bigcache 已经分成了 1024 分片，避免了大量锁竞争问题，又因为删除的多是已知存在需删除的场景，预检查可能是徒劳的，虽然 cache 是读多写少的场景，但还是怀疑其优化是否有效。

GC 垃圾回收的设计
----------

bigcache 的 gc 垃圾回收是指淘汰清理过期数据，其实现很简单，从头到尾遍历数据，看到过期就淘汰，不过期就中断 gc 任务。

需要注意的是 bigcache 的过期时间为固定的，不像 redis、ristretto 可随意配置不同的过期时间。bigcache 是按照 fifo 先进先出来存储的，所以先入 ringbuffer 的对象必然要比后进来的先淘汰。

`bigcache` 实例化 cache 对象时，会启动一个协程来进行垃圾回收。每隔一段时间进行一次垃圾回收，默认时长为 1 秒。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

下面为垃圾回收的具体的实现，由于 bigcache 预设了 1024 个数据分片，那么进行垃圾回收时，自然需要遍历每个 shard 分片。清理的原理如下。

1.  从 ringbuffer 的 head 头部指针获取最先入队的对象，这里是 get，不是 pop 操作 ;
2.  从数据中获取数据的插入时间，通过 `( 当前的时间 - 写入时间 ) > lifeWindow` 判断是否过期 ;
3.  如过期，则把 ringbufer 中头部的数据，也就是最老的数据干掉 ;
4.  如没过期，则直接中断本地的垃圾回收任务。

简单说，每次判断 shard ringbuffer 中最老数据是否过期，过期则删掉，直到遍历到不过期的数据为止。这里的删除其实移动了 ringbuffer 的 header 偏移量位置指针。

`lifeWindow` 参数来控制过期的时长，当太小时，缓存数据很快会被 gc 清理掉。bigcache 的 delete 只是标记删除，数据依然在 ringbuffer 中。

那么被删除的数据什么时候会被清理掉，这里有两个时机。

*   写操作，当 ringbuffer 已满，又无法扩容时，则先删掉老数据，再把新数据写进去。
*   垃圾回收，从头开始进行过期判断，只要过期就清理。当满足 ringbuffer 的空闲阈值时，ringbuffer 也会回收空间。

`isExpired` 用来判断是否过期，当 `( 当前的时间 - 写入时间 ) > lifeWindow` 时，说明该 entry 已过期。

总结
--

`bigcache` 是 golang 编写的高性能的缓存库，其设计很巧妙，通过数据分片解决高并发下锁竞争的问题，通过把数据存到 ringbuffer 来规避 golang gc 的开销。

但 `bigcache` 不太适合业务上的缓存对象，原因有两个。

其一 bigcache 不支持 lru / lfu 这类缓存淘汰算法，而使用 fifo 淘汰旧数据，这样势必影响缓存命中率和缓存效果。

其二 bigcache 不能支持除了 `[]byte` 以外的数据结构，毕竟业务上的对象多为自定义 `struct`。大家存取时不能每次都进行 encode、decode 编解码吧？毕竟使用社区中有一堆黑科技的 sonic json 库，序列化小对象也至少几十个 us，反序列化更是序列化的两倍。

个人更推荐使用 `ristretto` 构建业务的进程内缓存，其实现原理文章如下。

[golang ristretto 高性能缓存的设计实现原理](https://github.com/rfyiamcool/notes/blob/main/golang_ristretto_cache.md)

* * *

大家觉得文章对你有些作用！ 如果想赏钱，可以用微信扫描下面的二维码，感谢!  
另外再次标注博客原地址  [xiaorui.cc](http://xiaorui.cc/)  

![](https://raw.githubusercontent.com/lslz627/PicGo/master/weixin_new.jpg)