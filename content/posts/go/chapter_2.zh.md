---
layout: post
title: "Map 的实现原理"
description: "深入分析Go语言 Map 数据结构的实现原理"
date: 2025-02-19T10:39:14+08:00
image: "/posts/go/images/chapter_2-cover.png"
tags: ["Go"]
---

哈希表是一种非常常见的数据结构，在不同的编程语言中有不同的称呼，如字典、映射，而在Go中，将其称为`map`。

## 设计原理

哈希表拥有O(1)的读写性能，提供了键值之间的映射。而要实现一个哈希表，需要注意两点——哈希函数和哈希冲突的解决方法。

### 哈希函数

哈希函数是一种将任意长度的输入映射为固定长度输出的函数。理想状态下，哈希函数应该将不同的输入映射为不同的输出，这要求哈希函数的输出范围大于输入范围，但这是不可能的，不同输入映射为相同输出的情况，就被称为哈希冲突。比较实际的方式是让哈希函数的结果尽量地均匀分布，以尽可能地减少哈希冲突的发生，再结合工程手段解决哈希冲突问题。因此，要想实现性能优异的哈希表，哈希函数的选择是至关重要的。

<div class="content-image">
<img src="/posts/go/images/chapter_2-hash.png" width="50%">
<p>Hash Function</p>
</div>

在Go语言中，哈希函数的选择是动态的，由输入键的类型决定，如键是`string`类型时，使用的是<a href="https://zh.wikipedia.org/wiki/Murmur%E5%93%88%E5%B8%8C">`MurmurHash`</a>等高效哈希函数，而键是整型时，则通过简单的位运算进行哈希。并且在运行时，都会生成随机的哈希种子，使得相同的键在每次程序运行时，生成的哈希值也是不同的，这样做可以有效地规避哈希碰撞攻击（Hash Collision Attack）。

```
哈希碰撞攻击（Hash Collision Attack）是一种针对哈希函数或基于哈希的数据结构（如哈希表）的攻击方式。攻击者通过精心构造大量具有相同哈希值的输入，使得哈希表的性能急剧下降，甚至导致服务不可用。
```

### 哈希冲突

由上文可知，在实际情况下，即使使用了完美的哈希函数，也无法避免哈希冲突的问题，这时候就不得不使用工程手段来解决了，常用的解决方法有开放寻址法、拉链法（链表法、链地址法）、再哈希法（再散列法）。

#### 开放寻址法

开放寻址法的核心思想：当发生哈希冲突时，通过某种探测方法在哈希表中寻找下一个空闲的位置。

向哈希表插入数据，当我们通过

<p>
<math display="block"> <mi> n(key) = hash(key) % table_size </mi> </math>
<p>

计算出键的初始位置后，若初始位置已存在键值对且与插入键不相同时，则继续通过探测方法来寻找下一个索引不为空的位置，相同则更新值。

<div class="content-image">
<img src="/posts/go/images/chapter_2-hash-insert.png" width="60%">
<p>Insert</p>
</div>

常用的探测方法有

* 线性探测

<p>
<math display="block"> <mi> n(key, i) = ( hash(key) + i ) % table_size </mi> </math>
<p>

* 二次探测

<p>
<math display="block"><mrow> <mi> n(key, i) </mi> </mrow><mo> = </mo> <mrow> <mo> ( </mo> <mi> hash(key) </mi> <mo> + </mo> <msub> <mi> c </mi> <mn> 1 </mn> </msub> <mo> * </mo> <mi> i </mi> <mo> + </mo> <msub> <mi> c </mi> <mn> 2 </mn> </msub> <mo> * </mo> <msup> <mi> i </mi> <mn> 2 </mn> </msup> <mo> ) </mo> </mrow> <mo> % </mo> <mrow> <mi> table_size </mi> </mrow> <mrow> <mi> (c1,c2 ∊ C) </mi> </mrow> </math>
<p>

* 双重哈希

<p>
<math display="block"> <mi> n(key, i) = (hash1(key) + i * hash2(key)) % table_size </mi> </math>
<p>

当进行查询时，使用相同的方法计算键的初始位置，比较初始位置的键与查询键是否相同，不同则继续使用探测方法计算，直到找到要查询的键，或计算出索引为空的位置，这说明查询的键不存在。

<div class="content-image">
<img src="/posts/go/images/chapter_2-hash-search.png" width="60%">
<p>Search</p>
</div>

当进行删除时，开放寻址法无法真正删除元素，而是将要删除的元素标记为已删除，因为一旦真正删除了某个元素，就可能会导致探测方法被中断，使得寻址失效。

<div class="content-image">
<img src="/posts/go/images/chapter_2-hash-delete.png" width="60%">
<p>Delete</p>
</div>

装载因子（num(key)/table_size）是衡量开放寻址法的重要指标，当装载因子超出70%之后，哈希表性能会急速下降，当到达100%时，哈希表则完全失效，此时读写哈希表都需要遍历所有元素，时间复杂度为O(n)。

#### 拉链法（链表法、链地址法）

拉链法是大多数编程语言实现哈希表的方法，通常使用数组和链表的组合，有些时候也会引入红黑树以优化性能。拉链法的核心思想：哈希表中的索引不再存储一个键值对，而是存储一个链表，所有哈希到同一位置的键值对都存储在这个链表中，一般称这个链表为桶（Bucket）。

向哈希表中插入数据，当我们通过

<p>
<math display="block"> <mi> b(key) = hash(key) % table_size </mi> </math>
<p>

为键值对选择好桶后，遍历桶中的键值对，若存在相同键则更新值，若不存在相同键，则在桶中的链表后追加键值对。

<div class="content-image">
<img src="/posts/go/images/chapter_2-hash-bucket-insert.png" width="60%">
<p>Insert</p>
</div>

当进行查询时，使用相同的方法计算查询键对应的桶，再遍历桶中的链表，查看被查询的键是否存在。

<div class="content-image">
<img src="/posts/go/images/chapter_2-hash-bucket-search.png" width="60%">
<p>Search</p>
</div>

当进行删除时，使用相同的方法计算删除键对应的桶，再遍历桶中的链表，进行链表元素的删除，无需像开放寻址法一样使用标记的方式。

<div class="content-image">
<img src="/posts/go/images/chapter_2-hash-bucket-delete.png" width="60%">
<p>Delete</p>
</div>

使用拉链法实现哈希表时，装载因子的计算方式是元素数量/桶数量，相较于开放寻址法，拉链法对装载因子的容忍度更高，即使超出100%，也能正常工作，但一般也会再装载因子过大时进行扩容，增加桶的数量，避免性能下降严重。

#### 再哈希法（再散列法）

再哈希法的核心思想：当发生哈希冲突时，使用另外的哈希函数重新计算，直到不再冲突。这种方法虽然实现简单，不过需要维护多个哈希函数，需要进行多次计算，并且一旦所有的哈希函数都无法避免哈希冲突，该方法就失效了。

<p>
<math display="block"><mrow><msub><mi>h</mi><mi>i</mi></msub></mrow><mrow><mi>(key)</mi></mrow> <mo>=</mo> <mrow><msub><mi>hash</mi><mi>i</mi></msub></mrow> <mrow><mi>(key)</mi></mrow> <mo>%</mo> <mrow><mi>table_size</mi></mrow> </math>
<p>

<div class="content-image">
<img src="/posts/go/images/chapter_2-hash-double.png" width="60%">
<p>Insert</p>
</div>

## Go Map

### 结构

Go语言的哈希表采用拉链法实现，但其底层实现并没有使用传统的链表，而是使用了特殊的数据结构，并且还引入了溢出桶的概念。

Go语言的哈希表的被定义为`hmap`，其中包含了哈希表的元素数量(count)，哈希种子(hash0)，桶指针(buckets)，溢出桶指针(extra)等信息。

```go
type hmap struct {
	count     int
	flags     uint8
	B         uint8
	noverflow uint16
	hash0     uint32

	buckets    unsafe.Pointer
	oldbuckets unsafe.Pointer
	nevacuate  uintptr
	clearSeq   uint64

	extra *mapextra
}
```

而无论是正常桶还是溢出桶，其结构都是`bmap`，其中包含了存储的键值对（keys,values），键的tophash缓存(tophash)，指向下一个溢出桶的指针(overflow)等信息。`bmap`的绝大多数字段并没有以代码的形式在Go源码中定义，这是因为`bmap`的结构是高度优化的，并且会根据键和值的类型动态调整其内存布局。通过结构也可以发现，一个`bmap`最多可以存储8个键值对，当`bmap`被存满后继续向其中写入数据，就会被写入到`overflow`溢出桶中。

```go
type bmap struct {
    tophash  [8]uint8
    keys     [8]keyType
    values   [8]valueType
    overflow *bmap
}
```

<div class="content-image">
<img src="/posts/go/images/chapter_2-hash-go-map.png" width="80%">
<p>Go Map</p>
</div>

### 常见操作

#### 访问

当对`map`进行访问时，单返回值和多返回值访问将分别调用`runtime.mapaccess1`和`runtime.mapaccess2`，而`runtime.mapaccess2`也仅比`runtime.mapaccess1`多返回了一个标识键是否存在的布尔值，此外Go语言也对一些常见键类型的访问做了优化，如当键为`int`类型时，将调用`runtime.mapaccess2_fast64`。以`runtime.mapaccess2_fast64`为例

* 如果`map`为空或元素数量为0，则直接返回零值和false。

* 判断`map`是否被并发读写，如果被并发读写则panic，由此可见`map`并不是并发安全的。

* 如果`map`只有一个桶，不需要通过计算hash值来定位，直接获取桶。

* 如果不止一个桶，计算hash值，确定桶序号。如果`map`正在扩容，则需要确定该桶是否已经迁移，如果没有则需要使用旧桶的序号，才能获取正确的桶。

* 通过遍历桶中的tophash，快速比较哈希值的高8位，如果tophash匹配，则进一步比较键值，若找到匹配的键，则返回值和true。

* 若没有找到匹配的键，则继续遍历溢出桶，如果所有相关桶都没有找到匹配的键，则返回零值和false。

```go
func main() {
	testMap := map[int]string{
		1: "A",
		2: "B",
		3: "C",
	}

	val, ok := testMap[1]
	if !ok {
		fmt.Println("key not found")
	}
	fmt.Println("val:", val)
}

...
LEAQ    type:map[int]string(SB), AX
LEAQ    main..autotmp_19+304(SP), BX
MOVL    $1, CX
PCDATA  $1, $0
CALL    runtime.mapaccess2_fast64(SB)
MOVQ    (AX), DX
MOVQ    8(AX), SI
...

func mapaccess2_fast64(t *maptype, h *hmap, key uint64) (unsafe.Pointer, bool) {
	if h == nil || h.count == 0 {
		return unsafe.Pointer(&zeroVal[0]), false
	}
	if h.flags&hashWriting != 0 {
		fatal("concurrent map read and map write")
	}
	var b *bmap
	if h.B == 0 {
		b = (*bmap)(h.buckets)
	} else {
		hash := t.Hasher(noescape(unsafe.Pointer(&key)), uintptr(h.hash0))
		m := bucketMask(h.B)
		b = (*bmap)(add(h.buckets, (hash&m)*uintptr(t.BucketSize)))
		if c := h.oldbuckets; c != nil {
			if !h.sameSizeGrow() {
				m >>= 1
			}
			oldb := (*bmap)(add(c, (hash&m)*uintptr(t.BucketSize)))
			if !evacuated(oldb) {
				b = oldb
			}
		}
	}
	for ; b != nil; b = b.overflow(t) {
		for i, k := uintptr(0), b.keys(); i < abi.OldMapBucketCount; i, k = i+1, add(k, 8) {
			if *(*uint64)(k) == key && !isEmpty(b.tophash[i]) {
				return add(unsafe.Pointer(b), dataOffset+abi.OldMapBucketCount*8+i*uintptr(t.ValueSize)), true
			}
		}
	}
	return unsafe.Pointer(&zeroVal[0]), false
}
```

#### 写入

当向`map`中写入键值对时，将调用`runtime.mapassign`函数，写入流程与访问流程大致相同，但需要特别注意并发读写检查和`map`扩容。在写入数据前，会先检查当前是否在进行扩容，如果在扩容中，则会主动调用`runtime.growWork`函数，对目标桶进行数据迁移，迁移完成后，再在新桶中执行写入操作。而在写入数据后，会检查负载因子和溢出桶数量来判断是否需要进行扩容。

```go
func main() {
	testMap := make(map[int]string)
	testMap[1] = "A"
}

...
LEAQ    type:map[int]string(SB), AX
LEAQ    main..autotmp_1+224(SP), BX
MOVL    $1, CX
PCDATA  $1, $0
CALL    runtime.mapassign_fast64(SB)
MOVQ    $1, 8(AX)
...

func mapassign_fast64(t *maptype, h *hmap, key uint64) unsafe.Pointer {
	if h == nil {
		panic(plainError("assignment to entry in nil map"))
	}

	if h.flags&hashWriting != 0 {
		fatal("concurrent map writes")
	}
	hash := t.Hasher(noescape(unsafe.Pointer(&key)), uintptr(h.hash0))

	h.flags ^= hashWriting

	if h.buckets == nil {
		h.buckets = newobject(t.Bucket)
	}

again:
	bucket := hash & bucketMask(h.B)
	if h.growing() {
		growWork_fast64(t, h, bucket)
	}
	b := (*bmap)(add(h.buckets, bucket*uintptr(t.BucketSize)))

	var insertb *bmap
	var inserti uintptr
	var insertk unsafe.Pointer

bucketloop:
	for {
		for i := uintptr(0); i < abi.OldMapBucketCount; i++ {
			if isEmpty(b.tophash[i]) {
				if insertb == nil {
					insertb = b
					inserti = i
				}
				if b.tophash[i] == emptyRest {
					break bucketloop
				}
				continue
			}
			k := *((*uint64)(add(unsafe.Pointer(b), dataOffset+i*8)))
			if k != key {
				continue
			}
			insertb = b
			inserti = i
			goto done
		}
		ovf := b.overflow(t)
		if ovf == nil {
			break
		}
		b = ovf
	}

	if !h.growing() && (overLoadFactor(h.count+1, h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {
		hashGrow(t, h)
		goto again
	}

	if insertb == nil {
		insertb = h.newoverflow(t, b)
		inserti = 0
	}
	insertb.tophash[inserti&(abi.OldMapBucketCount-1)] = tophash(hash)

	insertk = add(unsafe.Pointer(insertb), dataOffset+inserti*8)
	*(*uint64)(insertk) = key

	h.count++

done:
	elem := add(unsafe.Pointer(insertb), dataOffset+abi.OldMapBucketCount*8+inserti*uintptr(t.ValueSize))
	if h.flags&hashWriting == 0 {
		fatal("concurrent map writes")
	}
	h.flags &^= hashWriting
	return elem
}
```

#### 删除

删除键值对的逻辑与写入基本相同，并且也会在删除前检查是否在进行扩容，若正在扩容，则先将目标桶进行迁移，迁移完成后再针对新桶进行键值对的删除工作。删除操作将调用`runtime.mapdelete`函数，具体实现可见Go源码。

#### 扩容

在探讨向`map`写入的时候，已经看到触发扩容的因素有两种，一是负载因子过大，二是过多的溢出桶，在扩容时将调用`runtime.hasGrow`函数。

```go
func hashGrow(t *maptype, h *hmap) {
	bigger := uint8(1)
	if !overLoadFactor(h.count+1, h.B) {
		bigger = 0
		h.flags |= sameSizeGrow
	}
	oldbuckets := h.buckets
	newbuckets, nextOverflow := makeBucketArray(t, h.B+bigger, nil)

	flags := h.flags &^ (iterator | oldIterator)
	if h.flags&iterator != 0 {
		flags |= oldIterator
	}
	h.B += bigger
	h.flags = flags
	h.oldbuckets = oldbuckets
	h.buckets = newbuckets
	h.nevacuate = 0
	h.noverflow = 0

	if h.extra != nil && h.extra.overflow != nil {
		if h.extra.oldoverflow != nil {
			throw("oldoverflow is not nil")
		}
		h.extra.oldoverflow = h.extra.overflow
		h.extra.overflow = nil
	}
	if nextOverflow != nil {
		if h.extra == nil {
			h.extra = new(mapextra)
		}
		h.extra.nextOverflow = nextOverflow
	}
}
```

Go语言针对这两种情况进行的扩容操作是不一样的。

* 等量扩容

当由于溢出桶过多导致扩容时，会触发等量扩容，其目的是整理`map`的存储结构，减少溢出桶的数量，从而提升访问效率。因此在等量扩容时，新桶组与旧桶组的桶数量一致，只是通过重新哈希将键值对分配到新的桶中，来减少溢出桶的使用。

* 翻倍扩容

当由于负载因子过大导致扩容时，会触发翻倍扩容，此时新桶组的桶数量是旧桶组的两倍。其主要目的是为了降低负载因子，减少哈希冲突的概率，提升访问效率。

并且`map`的扩容是渐进式的，即使正在进行扩容操作，也不会堵塞对`map`的读写和删除操作，在介绍有关`map`的读写和删除操作的部分也可以看到针对正在扩容阶段的`map`的处理逻辑。

## 总结

* 哈希表是一种常见且重要的数据结构，要实现性能优异的哈希表需要注意两个方面，哈希函数的选择和哈希冲突的处理方式，哈希函数应该选择输出尽可能均匀分布的，解决哈希冲突的方法主要有开放寻址法、拉链法、重哈希法。

* Go语言哈希表采用拉链法实现，设计了特殊的桶结构`bmap`，并采用了溢出桶的策略来降低哈希表扩容的频率。

* Go语言哈希表的读写和删除操作访问逻辑类似，首先通过哈希函数和哈希种子计算哈希值，然后依次遍历正常桶和溢出桶。写入操作可能会触发哈希表的扩容。

* Go语言哈希表的扩容有两种情况，等量扩容和翻倍扩容。等量扩容由溢出桶过多触发，主要目的是通过重新哈希将键值对分配到新桶中，整理哈希表结构，减少溢出桶数量，降低溢出桶的使用频率来提升访问效率。翻倍扩容由负载因子过大触发，主要目的是通过增加桶数量来降低负载因子，减少哈希冲突的发生从而提升访问效率。