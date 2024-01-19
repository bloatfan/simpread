> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/476649222)

关于如何在一个方向或者一个领域进阶项目，总共分为 4 步。

1. 分析关键点

2. 博采众长

3. 压缩提炼

4. 动手

下面这个链接是我已分布式缓存举例的一个项目进阶过程。

[顾茄：分布式缓存之 gocache 代码通读（1）](https://zhuanlan.zhihu.com/p/465508094)

同样，本篇回答也继续带来 golang 缓存的相关框架通读。

前面我们已经看了缓存的介绍、相关知识。还有 gocache 和 bigcache 的代码通读。感兴趣的同学可以点击链接查看：

[顾茄：分布式缓存之 gocache 代码通读（1）](https://zhuanlan.zhihu.com/p/465508094)[顾茄：分布式缓存之 - bigcache 代码通读](https://zhuanlan.zhihu.com/p/465987539)

从阅读难度上看，freecache 略大于 bigcache 远大于 gocache，从性能上则是完全反过来。这也是性能优化的一个很重要的规律，牺牲可读性换来性能，因为更高的性能就涉及更多的底层相关优化，此时代码的可读性显而易见就会降低。

FreeCache 介绍
------------

链接：

[GitHub - coocood/freecache: A cache library for Go with zero GC overhead.](https://github.com/coocood/freecache)

### FreeCache 特性

直接搬运 github 上的介绍，主要包括大数据量，零 GC 压力，线程安全，近似 LRU 算法，严格的内存使用等等。后面也将从代码分析中说明 FreeCache 是如何实现这五点的。

*   Store hundreds of millions of entries
*   Zero GC overhead
*   High concurrent thread-safe access
*   Pure Go implementation
*   Expiration support
*   Nearly LRU algorithm
*   Strictly limited memory usage
*   Come with a toy server that supports a few basic Redis commands with pipeline
*   Iterator support

### FreeCache 数据结构

如果没有看到数据结构的设计，只看代码会非常的吃力。整个 FreeCache 的数据结构只有 256 个 Segement，里面包括存储键值对索引的 slotData，每个索引指针名称为 entryPtr。还有存储数据的 ringbuf，ringbuf 存储的每个数据包括 24 字节的数据头和数据的实际键值对。这意味着每个 segment 只有一个 slotData 切片和 ringbuf 的 data 切片，也就是说，整个数据结构只有 256*2 个指针，非常少的指针数量意味着极小的 gc 压力。

![](https://raw.githubusercontent.com/lslz627/PicGo/master/v2-e8df30915080da0f70896bb169743ec1_r.jpg)

结构体与方法分析
--------

### cache

cache 包括两个数组，一个 256 大小的段（segment）数组和对应的锁数组。**这里采用数组而不是指针，也是减少 GC 压力。**

再看 set 函数，实质上与之前学习过的 BIgCache 类似。都是先利用 hash（key）找到对应的 segments 位置，再锁住整个 segment 并调用 segment 的 Set 函数（此处锁的粒度还是比较大的。。）。

```
type Cache struct {
	locks    [segmentCount]sync.Mutex
	segments [segmentCount]segment
}

// Set sets a key, value and expiration for a cache entry and stores it in the cache.
// If the key is larger than 65535 or value is larger than 1/1024 of the cache size,
// the entry will not be written to the cache. expireSeconds <= 0 means no expire,
// but it can be evicted when cache is full.
func (cache *Cache) Set(key, value []byte, expireSeconds int) (err error) {
	hashVal := hashFunc(key)
	segID := hashVal & segmentAndOpVal
	cache.locks[segID].Lock()
	err = cache.segments[segID].set(key, value, hashVal, expireSeconds)
	cache.locks[segID].Unlock()
	return
}
```

### Segment

一个 segment 包含 256 个槽（slots），每个槽包含一个指向数据（entry）的指针数组。仔细看每个 segment 包含一个 ringbuf，这是存储实体数据的字段。slotsData 则是存储数据的指针。其他的都是些辅助统计字段。

```
// a segment contains 256 slots, a slot is an array of entry pointers ordered by hash16 value
// the entry can be looked up by hash value of the key.
type segment struct {
	rb            RingBuf // ring buffer that stores data
	segId         int
	_             uint32
	missCount     int64
	hitCount      int64
	entryCount    int64
	totalCount    int64      // number of entries in ring buffer, including deleted entries.
	totalTime     int64      // used to calculate least recent used entry.
	timer         Timer      // Timer giving current time
	totalEvacuate int64      // used for debug
	totalExpired  int64      // used for debug
	overwrites    int64      // used for debug
	touched       int64      // used for debug
	vacuumLen     int64      // up to vacuumLen, new data can be written without overwriting old data.
	slotLens      [256]int32 // The actual length for every slot.
	slotCap       int32      // max number of entry pointers a slot can hold.
	slotsData     []entryPtr // shared by all 256 slots
}

func (seg *segment) set(key, value []byte, hashVal uint64, expireSeconds int) (err error) {
	if len(key) > 65535 {
		return ErrLargeKey
	}
	maxKeyValLen := len(seg.rb.data)/4 - ENTRY_HDR_SIZE
	if len(key)+len(value) > maxKeyValLen {
		// Do not accept large entry.
		return ErrLargeEntry
	}
	now := seg.timer.Now()
	expireAt := uint32(0)
	if expireSeconds > 0 {
		expireAt = now + uint32(expireSeconds)
	}

	slotId := uint8(hashVal >> 8)
	hash16 := uint16(hashVal >> 16)
	slot := seg.getSlot(slotId)
	idx, match := seg.lookup(slot, hash16, key)

	var hdrBuf [ENTRY_HDR_SIZE]byte
	hdr := (*entryHdr)(unsafe.Pointer(&hdrBuf[0]))
	if match {
		matchedPtr := &slot[idx]
		seg.rb.ReadAt(hdrBuf[:], matchedPtr.offset)
		hdr.slotId = slotId
		hdr.hash16 = hash16
		hdr.keyLen = uint16(len(key))
		originAccessTime := hdr.accessTime
		hdr.accessTime = now
		hdr.expireAt = expireAt
		hdr.valLen = uint32(len(value))
		if hdr.valCap >= hdr.valLen {
			//in place overwrite
			atomic.AddInt64(&seg.totalTime, int64(hdr.accessTime)-int64(originAccessTime))
			seg.rb.WriteAt(hdrBuf[:], matchedPtr.offset)
			seg.rb.WriteAt(value, matchedPtr.offset+ENTRY_HDR_SIZE+int64(hdr.keyLen))
			atomic.AddInt64(&seg.overwrites, 1)
			return
		}
		// avoid unnecessary memory copy.
		seg.delEntryPtr(slotId, slot, idx)
		match = false
		// increase capacity and limit entry len.
		for hdr.valCap < hdr.valLen {
			hdr.valCap *= 2
		}
		if hdr.valCap > uint32(maxKeyValLen-len(key)) {
			hdr.valCap = uint32(maxKeyValLen - len(key))
		}
	} else {
		hdr.slotId = slotId
		hdr.hash16 = hash16
		hdr.keyLen = uint16(len(key))
		hdr.accessTime = now
		hdr.expireAt = expireAt
		hdr.valLen = uint32(len(value))
		hdr.valCap = uint32(len(value))
		if hdr.valCap == 0 { // avoid infinite loop when increasing capacity.
			hdr.valCap = 1
		}
	}

	entryLen := ENTRY_HDR_SIZE + int64(len(key)) + int64(hdr.valCap)
	slotModified := seg.evacuate(entryLen, slotId, now)
	if slotModified {
		// the slot has been modified during evacuation, we need to looked up for the 'idx' again.
		// otherwise there would be index out of bound error.
		slot = seg.getSlot(slotId)
		idx, match = seg.lookup(slot, hash16, key)
		// assert(match == false)
	}
	newOff := seg.rb.End()
	seg.insertEntryPtr(slotId, hash16, newOff, idx, hdr.keyLen)
	seg.rb.Write(hdrBuf[:])
	seg.rb.Write(key)
	seg.rb.Write(value)
	seg.rb.Skip(int64(hdr.valCap - hdr.valLen))
	atomic.AddInt64(&seg.totalTime, int64(now))
	atomic.AddInt64(&seg.totalCount, 1)
	seg.vacuumLen -= entryLen
	return
}
```

**set 中，值得关注的点包括：**

1.  slot := seg.getSlot(slotId)， 这里由于 slotsData 数组是所有槽共用的，因此需要根据槽容量和 slotID 定位到偏移值，取出对应的 entryPtr, 这里**三段式索引在通常语法上是不允许的，第三个表示最大能索引的值，可以理解为** seg.slotsData[start(slotOff) : end(slotOff+seg.slotLens[slotId]), capend(slotOff+seg.slotCap)]**。**

```
func (seg *segment) getSlot(slotId uint8) []entryPtr {
	slotOff := int32(slotId) * seg.slotCap
	return seg.slotsData[slotOff : slotOff+seg.slotLens[slotId] : slotOff+seg.slotCap]
}
```

2. 寻找位置。idx, match := seg.lookup(slot, hash16, key), 这里是为了找到 set 的 key-value 在一个槽内的位置。作者利用了二分查找，感兴趣的同学可以推演其中一些极端的情况，比如所有值都大于传入的这个 hash-key，或者都小于，或者都等于，推演后可以发现，逻辑上都没问题。

```
func (seg *segment) lookup(slot []entryPtr, hash16 uint16, key []byte) (idx int, match bool) {
	idx = entryPtrIdx(slot, hash16)
	for idx < len(slot) {
		ptr := &slot[idx]
		if ptr.hash16 != hash16 {
			break
		}
		match = int(ptr.keyLen) == len(key) && seg.rb.EqualAt(key, ptr.offset+ENTRY_HDR_SIZE)
		if match {
			return
		}
		idx++
	}
	return
}
```

3. 寻找空间。找到位置之后，我们还要注意目前这个数据的位置是否可以承载对应的空间，我们分几种情况讨论。

1.key 匹配，原先的 value 空间大于现在的，那么很简单。把新值拷贝到旧值的空间中即可。

![](https://raw.githubusercontent.com/lslz627/PicGo/master/v2-7ccdbbd21d27161be2cea22f5a382ade_r.jpg)

2. 原先的 value 空间不够现在的 value 放置，拿就得删除掉原有的 key-value，再把新值放入。

![](https://raw.githubusercontent.com/lslz627/PicGo/master/v2-2237d715ffdc6eee5f52e89204bfb71e_r.jpg)

3.key 不匹配，也是一样，我们从队尾把新值放入即可。整体的逻辑就如 evacuate 函数所示。

evacuate 函数, 可以理解成为新数据寻找空间，逻辑如下：1. 先判断剩余空间是否小于新数据大小，如果不小于说明剩余空间足够，那么就可以退出了。如果小于按以下优先级来腾出空间。

*   读取 segment 第一个数据（对应 offset 的），如果该数据已经删除，那么就更新 seg.vacuumLen
*   如果该数据已经过期或很久未使用或已经搜寻多次了，删除该数据并腾出空间。否则就继续往后找其他数据是否可以删除。

```
func (seg *segment) evacuate(entryLen int64, slotId uint8, now uint32) (slotModified bool) {
	var oldHdrBuf [ENTRY_HDR_SIZE]byte
	consecutiveEvacuate := 0
	for seg.vacuumLen < entryLen {
		oldOff := seg.rb.End() + seg.vacuumLen - seg.rb.Size()
		seg.rb.ReadAt(oldHdrBuf[:], oldOff)
		oldHdr := (*entryHdr)(unsafe.Pointer(&oldHdrBuf[0]))
		oldEntryLen := ENTRY_HDR_SIZE + int64(oldHdr.keyLen) + int64(oldHdr.valCap)
		if oldHdr.deleted {
			consecutiveEvacuate = 0
			atomic.AddInt64(&seg.totalTime, -int64(oldHdr.accessTime))
			atomic.AddInt64(&seg.totalCount, -1)
			seg.vacuumLen += oldEntryLen
			continue
		}
		expired := oldHdr.expireAt != 0 && oldHdr.expireAt < now
		leastRecentUsed := int64(oldHdr.accessTime)*atomic.LoadInt64(&seg.totalCount) <= atomic.LoadInt64(&seg.totalTime)
		if expired || leastRecentUsed || consecutiveEvacuate > 5 {
			seg.delEntryPtrByOffset(oldHdr.slotId, oldHdr.hash16, oldOff)
			if oldHdr.slotId == slotId {
				slotModified = true
			}
			consecutiveEvacuate = 0
			atomic.AddInt64(&seg.totalTime, -int64(oldHdr.accessTime))
			atomic.AddInt64(&seg.totalCount, -1)
			seg.vacuumLen += oldEntryLen
			if expired {
				atomic.AddInt64(&seg.totalExpired, 1)
			} else {
				atomic.AddInt64(&seg.totalEvacuate, 1)
			}
		} else {
			// evacuate an old entry that has been accessed recently for better cache hit rate.
			newOff := seg.rb.Evacuate(oldOff, int(oldEntryLen))
			seg.updateEntryPtr(oldHdr.slotId, oldHdr.hash16, oldOff, newOff)
			consecutiveEvacuate++
			atomic.AddInt64(&seg.totalEvacuate, 1)
		}
	}
	return
}
```

### entry 与 RingBuf

entryPtr 为每个数据的指针, 主要包括该 entry 在 ringbuf 中的位置、entry 的 hash 值，长度等等。entryHdr 则是跟在每个 key-value 数据后面的相关信息。

RingBuf 则包含着一些数据存储字段和起始位置等等。阅读 RingBuf 的代码时要注意：为了提高性能，ringbuf 内的空间是覆盖写的，也就是如果数据写满了，将会从开头的位置写入。

```
type entryPtr struct {
	offset   int64  // entry offset in ring buffer
	hash16   uint16 // entries are ordered by hash16 in a slot.
	keyLen   uint16 // used to compare a key
	reserved uint32
}

// 在ringbuf该数据的位置标记删除为true，并且删除掉该entryptr。
func (seg *segment) delEntryPtr(slotId uint8, slot []entryPtr, idx int) {
	offset := slot[idx].offset
	var entryHdrBuf [ENTRY_HDR_SIZE]byte
	seg.rb.ReadAt(entryHdrBuf[:], offset)
	entryHdr := (*entryHdr)(unsafe.Pointer(&entryHdrBuf[0]))
	entryHdr.deleted = true
	seg.rb.WriteAt(entryHdrBuf[:], offset)
	copy(slot[idx:], slot[idx+1:])
	seg.slotLens[slotId]--
	atomic.AddInt64(&seg.entryCount, -1)
}

// entry header struct in ring buffer, followed by key and value.
type entryHdr struct {
	accessTime uint32
	expireAt   uint32
	keyLen     uint16
	hash16     uint16
	valLen     uint32
	valCap     uint32
	deleted    bool
	slotId     uint8
	reserved   uint16
}


// Ring buffer has a fixed size, when data exceeds the
// size, old data will be overwritten by new data.
// It only contains the data in the stream from begin to end
type RingBuf struct {
	begin int64 // beginning offset of the data stream.
	end   int64 // ending offset of the data stream.
	data  []byte
	index int //range from '0' to 'len(rb.data)-1'
}


// read up to len(p), at off of the data stream.
func (rb *RingBuf) ReadAt(p []byte, off int64) (n int, err error) {
	if off > rb.end || off < rb.begin {
		err = ErrOutOfRange
		return
	}
	readOff := rb.getDataOff(off)
	readEnd := readOff + int(rb.end-off)
	if readEnd <= len(rb.data) {
		n = copy(p, rb.data[readOff:readEnd])
	} else {
		n = copy(p, rb.data[readOff:])
		if n < len(p) {
			n += copy(p[n:], rb.data[:readEnd-len(rb.data)])
		}
	}
	if n < len(p) {
		err = io.EOF
	}
	return
}
```

总结
--

在本篇中，我们梳理了 freeCache 的数据结构，有以下几点值得大家在代码中学习。

1.  指针的使用，指针就意味着 gc 负担，如果是非常高的性能场景，则尽量减少指针的使用。
2.  在 []byte 与 struct 的转化中大量使用 unsafe.Pointer 进行读取和转化，可以优化读取性能。
3.  由于是在并发场景下运行，为了保证数据操作的原子性，使用 atomic.AddInt64。
4.  hash 与二分查找等经典的算法仍然有很大的用武之地，无论在键值匹配还是在数据查找中都很有帮助。

最后祝大家金三银四春招顺利，我是顾茄，干杯！

![](https://raw.githubusercontent.com/lslz627/PicGo/master/v2-3ae73b404d71ba44da6c4fbbc6b4d5d3_r.jpg)