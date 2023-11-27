> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cyhone.com](https://www.cyhone.com/articles/bigcache/)

> 本文属于 《Golang 源码剖析系列》 Bigcache 是用 Golang 实现的本地内存缓存的开源库，主打的就是可缓存数据量大，查询速度快。

[Bigcache](https://github.com/allegro/bigcache) 是用 Golang 实现的本地内存缓存的开源库，主打的就是**可缓存数据量大，查询速度快。** 在其官方的介绍文章[《Writing a very fast cache service with millions of entries in Go》](https://blog.allegro.tech/2016/03/writing-fast-cache-service-in-go.html)一文中，明确提出了 bigcache 的设计目标：

1.  **多：** 缓存的元素数量非常大，可以达到百万级或千万级。
2.  **快：** 对延迟有非常高的要求，平均延迟要求在 5 毫秒以内。redis、memcached 之类的就不考虑在内了，毕竟用 Redis 还要多走一遍网络 IO。
3.  **稳：** 99.9 分位延迟应在 10 毫秒左右，99.999 分位延迟应在 400 毫秒左右。

目前有许多开源的 cache 库，大部分都是基于 map 实现的，例如 go-cache,ttl-cache 等。bigcache 明确指出，当数据量巨大时，直接基于 map 实现的 cache 库将出现严重的性能问题，这也是他们设计了一个全新的 cache 库的原因。

本文将通过分析 bigcache v3.1.0 的源码，揭秘 bigcache 如何解决现有 map 库的性能缺陷，以极致的性能优化，实现超高性能的缓存库。

[](#如何避免gc对map的影响)如何避免 GC 对 map 的影响
-----------------------------------

当 map 里面数据量非常大时，会出现性能瓶颈。这是因为在 Golang 进行 GC 时，会扫描 map 中的每个元素。当 map 足够大时，GC 时间过长，会对程序的性能造成巨大影响。

根据 bigcache 介绍文章的测试，在缓存数据达到数百万条时，接口的 99th 百分位延迟超过了一秒。监测指标显示堆中超过 4,000 万个对象，GC 的标记和扫描阶段耗时超过了四秒。这样的延迟对于 bigcache 来说是完全无法接受的。

这个问题在 Go 1.5 版本中有一项专门的优化 ([issue-9477](https://github.com/golang/go/issues/9477))：如果 map 的 key 和 value 中使用没有指针，那么 GC 时将无需遍历 map。例如`map[int]int`、`map[int]bool`。这是当时的 pull request: [go-review.googlesource.com/c/go/+/3288](https://go-review.googlesource.com/c/go/+/3288)。里面提到：

> Currently scanning of a map[int]int with 2e8 entries (~8GB heap)  
> takes ~8 seconds. With this change scanning takes negligible time.

对 2e8 个元素的 map[int]int 上进行了测试，GC 扫描时间从 8 秒减少到 0。

为什么当 map 的 key 和 value 不包含指针时，可以省去对元素的遍历扫描呢？这是因为 map 中的 int、bool 这种不可能会和外部变量有引用关系：

1.  int、bool 这种在 map 中存储的就是值本身。
2.  map 的 key 和 value 不可被寻址。也就是说，以`map[int]int`为例，外部没有办法取到这个 key 和 value 的指针，那也就无从引用了。

这个优化听起来非常强大好用，但是在 Golang 中指针无处不见，结构体指针、切片甚至字符串的底层实现都包含指针。一旦在 map 中使用它们（例如 map[int][]byte、map[string]int），同样会触发垃圾回收器的遍历扫描。

[](#bigcache的整体设计)bigcache 的整体设计
--------------------------------

bigcache 整体设计的出发点都是基于上文提到的 Golang 对 Map GC 优化，整个设计思路包含几个方面：

1.  数据分片存储，以降低锁冲突并提升并发量。
2.  避免在 map 中存储指针，从而避免在 GC 时对 map 进行遍历扫描。
3.  采用 FIFO 式的 Ring Buffer 设计，简化整体内存设计逻辑。

![](https://raw.githubusercontent.com/lslz627/PicGo/master/bigcache.png)

### [](#数据分shard)数据分 shard

这是一个非常常见的数据存储优化手段。表面上 bigcache 中所有的数据是存在一个大 cache 里面，但实际上底层数据分成了 N 个不互重合的部分，每一个部分称为一个 shard。

在 Set 或者 Get 数据时，先对 key 计算 hash 值，根据 hash 值取余得到目标 shard，之后所有的读写操作都是在各自的 shard 上进行。

以 Set 方法为例：

```
func (c *BigCache) Set(key string, entry []byte) error {
	hashedKey := c.hash.Sum64(key)
	shard := c.getShard(hashedKey)
	return shard.set(key, hashedKey, entry)
}
```

这么做的优势是可以减少锁冲突，提升并发量：当一个 shard 被加上 Lock 的时候，其他 shard 的读写不受影响。

在 bigcache 的设计中，对于 shard 有如下要求：

1.  一旦建好，shard 将不改变。这带来的两点好处：
    *   不用再考虑 shard 变化时的数据迁移问题。
    *   因为 shard 数组是固定不变的，因此从 shard 数组中根据 hash 值取目标 shard 的时候，就无需加锁了。
2.  shard 个数必须是 2 的平方数。这么做的好处是，对 2 的平方数取余可以改成位运算，会比传统的`%`快很多（根据不权威的 benchmark，计算速度大概会有 2 倍左右的差距）。

```
func (c *BigCache) getShard(hashedKey uint64) (shard *cacheShard) {
    // shardMask:  uint64(config.Shards - 1)
	return c.shards[hashedKey&c.shardMask]
}
```

3.  bigcache 的 shard 数默认值是 1024。

### [](#map不存原始数据避免gc遍历扫描)map 不存原始数据，避免 GC 遍历扫描

前文提到，map 的 key 和 value 一旦涉及指针相关的类型，GC 时就会触发遍历扫描。

因此在 bigcache 的设计中，shard 中的 map 直接定义为了`map[uint64]uint32` ，避免了存储任何指针。shard 的结构体定义如下:

```
type cacheShard struct {
	...
	hashmap     map[uint64]uint32
	entries     queue.BytesQueue
	...
}
```

其中：`hashmap`的 key 是 cache key 的 hash 值，而 value 仅仅是个 uint32。这显然不是我们 Set 的时候 value 的原始 byte 数组。

那 value 的原始值存在了哪里？答案是 cacheShard 中的另外一个属性`entries queue.BytesQueue`。  
`queue.BytesQueue`是一个 ring buffer 的内存结构，本质上就是个超大的`[]byte`数组，里面存放了所有的原始数据。每个原始数据就存放在这个大 []byte 数组中的其中一段。

hashmap 中 uint32 的 value 值存放的就是 value 的原始值在`BytesQueue`中的数组下标。(其实并不只是原始的 value 值，里面也包含了 key、插入时间戳等信息）

![](https://raw.githubusercontent.com/lslz627/PicGo/master/entry.png)

之所以用一个大的`[]byte`数组和 ring buffer 结构，除了方便管理和复用内存之外，一个更重要的原因是：对于 []byte 数组, GC 时只用看做一个变量扫描，无需再遍历全部数组。这样又避免了海量数据对 GC 造成的负担。

### [](#fifo式的内存结构设计)FIFO 式的内存结构设计

bigcache 在内存结构设计上完全遵循 FIFO 原则：

1.  新增数据，包括对老数据的修改，都是直接 Append 新数据到`BytesQueue`中。基本不直接对内存进行修改和删除等。
2.  每个数据项不可以定制单独的缓存时长，必须全部保持一致。这对数据淘汰非常友好，下文会详细讲述。

这样一整套设计约定下来，bigcache 的逻辑变成非常简洁明了，但这样同时造成了 bigcache 的局限性。

```
cache.Set("my-unique-key", []byte("value"))
```

前面讲述了 bigcache 的设计思想之后，Set 的整个逻辑也就很清晰了：

1.  计算 key 的 hash 值，得到对应的 shard
2.  将 key 和 value 等信息序列化成指定格式的 []byte, push 到 BytesQueue 中。
3.  根据 BytesQueue 返回的内存偏移量 (也就是数组下标)，将 key(hash 值) 和 value(数组下标)设置 hashmap 中。

这里需要注意的是，在 bigcache 的设计里面，Set 时 value 一定得是个`[]byte`类型。

前文讲到，bigcache 中所有的原始数据都会被塞到一个大的 []byte 数组里。因此对于 bigcache 来说最理想的肯定是直接给到`[]byte`最为方便，否则还需要考虑序列化等问题。

BytesQueue 是一个 ring buffer 设计，本文不再细究其实现了，和其他 ring buffer 的结构大同小异。

除了正常的 set 逻辑外，还有一些额外的情况需要考虑在内：

**情况 1：如果 key 之前设置过，Set 的时候会如何处理？**

在其他 cache 库的实现中，这种情况一般是找到旧值、删除，然后把新值设置到旧值的位置。

但在 bigcache 中并不是这样，前文提到，bigcache 的内存结构设计是 FIFO 式的，哪怕是有旧值的情况下，新值也不会复用其内存，依旧是 push 新的 value 到队列中。

那旧值将如何处理的呢？我们看下代码：

```
if previousIndex := s.hashmap[hashedKey]; previousIndex != 0 {
		if previousEntry, err := s.entries.Get(int(previousIndex)); err == nil {
			resetHashFromEntry(previousEntry)
			//remove hashkey
			delete(s.hashmap, hashedKey)
		}
	}
```

最核心的一句就是：`delete(s.hashmap, hashedKey)`

简单来说：**之前的旧值并未从内存中移除，仅仅只是将其偏移量从 s.hashmap 中移除了，使得外部读不到。**

那旧值什么时候会被淘汰呢？会有两种情况：

1.  如果设置了`CleanWindow` ，且旧值刚好过时，会被清理的定时器自动淘汰
2.  如果设置了`MaxEntrySize` 或者`HardMaxCacheSize`，当内存满时，也会触发最旧数据的淘汰。

在此之前，旧值的数据一直都会保留在内存中。

另外还有`resetHashFromEntry` ，这个逻辑主要是把 entry 中的 hash 部分的数值置为 0。这么做只是打上一个已处理的标记，保证数据在淘汰的时候不再去调用 OnRemove 的 callback 而已。

其实这里还有个场景：当`s.hashmap[hashedKey]`存在 value 时，并不一定是设置过这个 key，也有可能发生了 **hash 碰撞**。

按照上述逻辑，bigcache 并未对 hash 碰撞做特殊处理，统一都把之前相同 hash 的旧 key 删除。 毕竟这只是缓存的场景，并不保证之前 Set 进去的数据一直会存在。

**问题 2：当 ring buffer 满时，无法继续 push 数据，bigcache 会如何处理？**

情况分成两种：

*   如果`entries queue.BytesQueue` 未达到设定的 HardMaxCacheSize（最大内存上限），或者无 HardMaxCacheSize 要求，则直接扩容`queue.BytesQueue` 直到达到上限。不过扩容的时候，是创建了一个新的空`[]byte`数组，把原有数据 copy 过去。
*   如果内存已达上限，无法继续扩容，则会尝试删除最旧数据（无论是否过期），直至可以将数据放到`BytesQueue`中。如果这个时候新数据非常大，可能会为此淘汰掉许多旧数据。

```
entry, _ := cache.Get("my-unique-key")
fmt.Println(string(entry))
```

Get 基本上是 Set 的逆过程，整个过程更简单一些，没有太多额外的知识可讲。不过在使用时，需要注意的是：

*   Get 时如果数据到达了过期时间，但暂时还没有被清掉，这个时候也能正常查到 value，不会报错。 其实这个倒是符合大多数的实际需求场景，实际场景中其实对缓存过期时间并没有那么敏感，短时间读到旧值一般都是可以接受的。
*   如果对于缓存时间敏感的场景，可以使用 GetWithInfo 接口，返回值中有是否过期的标识。

跟删除有关的核心逻辑只有这两行，整个逻辑和 Set 过程中清除旧值的一样:

```
...
delete(s.hashmap, hashedKey)
...
resetHashFromEntry(wrappedEntry)
...
```

不过在调用`bigcache.Delete`接口时需要注意的是，如果 key 不存在时，会返回一个`ErrEntryNotFound`

上面讲到删除逻辑和 set 时清除旧值时，都只是简单的把 key 从 map 中删除，不让外部读取到而已。那原始值什么时候删呢？答案就是过期淘汰。

bigcache 有个设计上的优势：bigcache 没有开放单个元素的可过期时间，所有元素的 cache 时长都是一样的，这就意味着**所有元素的过期时间在队列中天然有序。**

这就使得淘汰逻辑非常简单，代码如下：

```
func (s *cacheShard) cleanUp(currentTimestamp uint64) {
	s.lock.Lock()
	for {
		if oldestEntry, err := s.entries.Peek(); err != nil {
			break
		} else if evicted := s.onEvict(oldestEntry, currentTimestamp, s.removeOldestEntry); !evicted {
			break
		}
	}
	s.lock.Unlock()
}
```

其实就是从头到尾遍历数组，直至元素不过期就跳出。

另外，即使淘汰过期数据时，数据也并未被真实的删除，仅仅对应于 ring buffer 中 head 和 tail 下标的移动。

这样整个删除过程非常轻量级，好处不仅在于逻辑更简单，更重要的是，淘汰时需要对整个 shard 加写锁，这种对有序数组的遍历删除，加锁的时间会非常短（当然也取决于这个时刻过期的数据条数）。

当然，这也意味着 bigcache 的局限性：数据过期模式非常简单，很多场景我们想做 LRU 式的缓存逻辑也无法办到。

此外从这里可以得知，哪怕是经过了淘汰，bigcache 的内存也不会主动降下去，除非外部调用了 Reset 方法。因此在实际实践中，我们最好是控制好 HardMaxCacheSize，以免 OOM。

bigcache 的主要逻辑已经基本讲完了，作为一个以性能为卖点的 cache 库，bigcache 在细节上也有大量的性能优化：

*   **varint 的使用：** 在最开始讲 bigcache 中每个 entry 结构的设计时，图中有一个 blocksize，代表数据 entry 的大小，用于 bigcache 确定数据边界。这里 blocksize 用到了 varint 来表示，可以一定程度上减小数据量。具体 varint 的介绍可以参考我的另外一篇文章[《解读 Golang 标准库里的 varint 实现》](https://www.cyhone.com/articles/golang-varint/)。
    
*   **buffer 内存复用**：在每次 set 数据的时候，上面 varint 和整个 entry 都需要动态地分配内存，bigcache 这里在每个 shard 中内置了两个全局的 buffer： `headerBuffer` 和`entrybuffer` ，避免了每次的内存分配。
    
*   **自己实现 fnv Hash：** bigcache 自己实现了一套 [fnv hash](https://github.com/allegro/bigcache/blob/main/fnv.go)，并没有用 go 官方标准库的，这也是基于性能的考虑。在 Go 官方的实现中 [hash/fnv/fnv.go](https://github.com/golang/go/blob/master/src/hash/fnv/fnv.go#L50)，创建 Fnv 对象的时候，有这么一段逻辑：
    
    ```
    func New32a() hash.Hash32 {
    	var s sum32a = offset32
    	return &s
    }
    ```
    
    根据 Golang 的逃逸分析，s 这个变量在结束的时候会被外部用到，这样 Go 编译器会将其分配到堆上（逃逸到堆上）。
    
    我们知道，直接在栈上操作内存比堆上更快速，因此 bigcache 实现了一个基于栈内存的 fnv hash 版本。
    

bigcache 的介绍文章中也提到，JSON 序列化问题成为了一个性能问题：

> While profiling our application, we found that the program spent a huge amount of time on JSON deserialization. Memory profiler also reported that a huge amount of data was processed by `json.Marshal`.

他们换成了 ffjson 来替换 go 标准库中的 json 操作，性能得到了不少的提升。

不过这样给我们提了个醒，如果不是海量数据，尚未达到 map 的 gc 瓶颈，倒是没有必要直接就上 bigcache。

`bigcache.Config`中有很多配置参数，这里大概列一下：

```
// Config for BigCache
type Config struct {
	// Number of cache shards, value must be a power of two
	// shard个数。必须2的平方数。
	Shards int
	// Time after which entry can be evicted
  // 最小粒度是秒，当CleanWindow设置的时候，一定要设置这个值
	LifeWindow time.Duration
	// Interval between removing expired entries (clean up).
	// If set to <= 0 then no action is performed. Setting to < 1 second is counterproductive — bigcache has a one second resolution.
	// 如果没有设置，数据将不会被定时清理。最好大于1秒，因为bigcache的最小时间粒度就是秒
	CleanWindow time.Duration
	// Max number of entries in life window. Used only to calculate initial size for cache shards.
	// When proper value is set then additional memory allocation does not occur.
	MaxEntriesInWindow int
	// Max size of entry in bytes. Used only to calculate initial size for cache shards. 
  // 单条数据最大的size，并不会做强制约束，只是用来初始化cache大小用，这个是仅包含用户自己设置的key和value的大小。
	MaxEntrySize int
	// StatsEnabled if true calculate the number of times a cached resource was requested.
	// 是否对每条数据都开启hit次数统计的功能
	StatsEnabled bool
	// Verbose mode prints information about new memory allocation
	Verbose bool
	// Hasher used to map between string keys and unsigned 64bit integers, by default fnv64 hashing is used.
 // hash函数，默认是bigcache自己实现的fnv
	Hasher Hasher
	// HardMaxCacheSize is a limit for BytesQueue size in MB.
	// It can protect application from consuming all available memory on machine, therefore from running OOM Killer.
	// Default value is 0 which means unlimited size. When the limit is higher than 0 and reached then
	// the oldest entries are overridden for the new ones. The max memory consumption will be bigger than
	// HardMaxCacheSize due to Shards' s additional memory. Every Shard consumes additional memory for map of keys
	// and statistics (map[uint64]uint32) the size of this map is equal to number of entries in
	// cache ~ 2×(64+32)×n bits + overhead or map itself.
 // 最大内存数限制。
	HardMaxCacheSize int
	// OnRemove is a callback fired when the oldest entry is removed because of its expiration time or no space left
	// for the new entry, or because delete was called.
	// Default value is nil which means no callback and it prevents from unwrapping the oldest entry.
	// ignored if OnRemoveWithMetadata is specified.
	OnRemove func(key string, entry []byte)
	// OnRemoveWithMetadata is a callback fired when the oldest entry is removed because of its expiration time or no space left
	// for the new entry, or because delete was called. A structure representing details about that specific entry.
	// Default value is nil which means no callback and it prevents from unwrapping the oldest entry.
	OnRemoveWithMetadata func(key string, entry []byte, keyMetadata Metadata)
	// OnRemoveWithReason is a callback fired when the oldest entry is removed because of its expiration time or no space left
	// for the new entry, or because delete was called. A constant representing the reason will be passed through.
	// Default value is nil which means no callback and it prevents from unwrapping the oldest entry.
	// Ignored if OnRemove is specified.
	OnRemoveWithReason func(key string, entry []byte, reason RemoveReason)

	// Logger is a logging interface and used in combination with `Verbose`
	// Defaults to `DefaultLogger()`
	Logger Logger
}
```