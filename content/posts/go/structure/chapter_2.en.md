---
layout: post
title: "The Implementation Principle of Map"
description: "An in-depth analysis of the implementation principle of the Map data structure in the Go language."
date: 2025-02-19T10:39:10+08:00
image: "/posts/go/structure/images/chapter_2-cover.jpg"
tags: ["Go"]
---

Hash tables are a very common data structure and have different names in various programming languages, such as dictionaries or maps. In Go, they are called `map`.

## Design Principle

Hash tables offer O(1) read and write performance and provide a mapping between keys and values. To implement a hash table, two points need to be noted - the hash function and the solution to hash collisions.

### Hash Function

A hash function is a function that maps inputs of arbitrary length to outputs of a fixed length. Ideally, a hash function should map different inputs to different outputs, which requires the output range of the hash function to be larger than the input range. However, this is impossible, and the situation where different inputs are mapped to the same output is called a hash collision. A more practical approach is to make the results of the hash function as evenly distributed as possible to minimize the occurrence of hash collisions, and then use engineering methods to solve the hash collision problem. Therefore, the selection of a hash function is crucial for achieving a high-performance hash table.

<div class="content-image">
<img src="/posts/go/structure/images/chapter_2-hash.png" width="50%">
<p>Hash Function</p>
</div>

In Go language, the choice of hash function is dynamic and determined by the type of the input key. For instance, when the key is of type `string`, efficient hash functions like <a href="https://zh.wikipedia.org/wiki/Murmur%E5%93%88%E5%B8%8C">`MurmurHash`</a> are used, while for integer keys, a simple bit operation is employed for hashing. Additionally, at runtime, a random hash seed is generated, ensuring that the same key produces different hash values each time the program runs. This approach effectively mitigates hash collision attacks.

```
A hash collision attack is a type of attack targeting hash functions or hash-based data structures such as hash tables. Attackers deliberately construct a large number of inputs with the same hash value, causing the performance of the hash table to sharply decline and even making the service unavailable.
```

### Hash Collision

As mentioned above, in practical situations, even if a perfect hash function is used, hash collisions cannot be avoided. At this point, engineering methods must be employed to solve the problem. Common solutions include <i>Open Addressing</i>, <i>Open Hashing</i>, and <i>Rehashing</i>.

#### Open Addressing

The core idea of <i>Open Addressing</i> is when a hash collision occurs, a certain probing method is used to find the next available position in the hash table.

Inserting data into a hash table, when we do so through 

<p>
<math display="block"> <mi> n(key) = hash(key) % table_size </mi> </math>
<p>

After calculating the position of the key, if the position already has a key-value pair and it is not the same as the key being inserted, then continue to search for the next empty index position through the probing method; if they are the same, update the value.

<div class="content-image">
<img src="/posts/go/structure/images/chapter_2-hash-insert.png" width="60%">
<p>Insert</p>
</div>

Commonly used probing methods include

* Linear Probing

<p>
<math display="block"> <mi> n(key, i) = ( hash(key) + i ) % table_size </mi> </math>
<p>

* Quadratic Probing

<p>
<math display="block"><mrow> <mi> n(key, i) </mi> </mrow><mo> = </mo> <mrow> <mo> ( </mo> <mi> hash(key) </mi> <mo> + </mo> <msub> <mi> c </mi> <mn> 1 </mn> </msub> <mo> * </mo> <mi> i </mi> <mo> + </mo> <msub> <mi> c </mi> <mn> 2 </mn> </msub> <mo> * </mo> <msup> <mi> i </mi> <mn> 2 </mn> </msup> <mo> ) </mo> </mrow> <mo> % </mo> <mrow> <mi> table_size </mi> </mrow> <mrow> <mi> (c1,c2 ∊ C) </mi> </mrow> </math>
<p>

* Double Hashing

<p>
<math display="block"> <mi> n(key, i) = (hash1(key) + i * hash2(key)) % table_size </mi> </math>
<p>

When performing a query, the same method is used to calculate the position of the key. The key at the position is compared with the query key. If they are not the same, the probing method is continued to calculate until the key to be queried is found or an empty position in the index is calculated, which indicates that the queried key does not exist.

<div class="content-image">
<img src="/posts/go/structure/images/chapter_2-hash-search.png" width="60%">
<p>Search</p>
</div>

When performing deletion, <i>Open Addressing</i> cannot truly delete an element but marks the element to be deleted as deleted. This is because once an element is truly deleted, it may cause the probing method to be interrupted, resulting in the failure of addressing.

<div class="content-image">
<img src="/posts/go/structure/images/chapter_2-hash-delete.png" width="60%">
<p>Delete</p>
</div>

The load factor (num(key)/table_size) is an important metric for evaluating <i>Open Addressing</i>. When the load factor exceeds 70%, the performance of the hash table will drop sharply. When it reaches 100%, the hash table becomes completely ineffective. At this point, reading and writing to the hash table require traversing all elements, with a time complexity of O(n).

#### Open Hashing

<i>Open Hashing</i> is the method used by most programming languages to implement hash tables, typically combining arrays and linked lists. Sometimes, red-black trees are introduced to optimize performance. The core idea of <i>Open Hashing</i> is that the index in the hash table no longer stores a single key-value pair but a linked list. All key-value pairs that hash to the same position are stored in this linked list, which is generally referred to as a bucket.

Inserting data into a hash table, when we do so through

<p>
<math display="block"> <mi> b(key) = hash(key) % table_size </mi> </math>
<p>

After choosing the appropriate bucket for the key-value pair, traverse the key-value pairs in the bucket. If a key is found to be the same, update the value. If no identical key is found, append the key-value pair to the end of the linked list in the bucket.

<div class="content-image">
<img src="/posts/go/structure/images/chapter_2-hash-bucket-insert.png" width="60%">
<p>Insert</p>
</div>

When performing a query, the same method is used to calculate the bucket corresponding to the query key, and then the linked list in the bucket is traversed to check if the queried key exists.

<div class="content-image">
<img src="/posts/go/structure/images/chapter_2-hash-bucket-search.png" width="60%">
<p>Search</p>
</div>

When performing deletion, the same method is used to calculate the bucket corresponding to the deletion key, and then the linked list in the bucket is traversed to delete the linked list element. There is no need to use the marking method as in <i>Open Addressing</i>.

<div class="content-image">
<img src="/posts/go/structure/images/chapter_2-hash-bucket-delete.png" width="60%">
<p>Delete</p>
</div>

When implementing a hash table using <i>Open Hashing</i>, the load factor is calculated as the number of elements divided by the number of buckets. Compared to <i>Open Addressing</i>, the method has a higher tolerance for the load factor. It can still function normally even when it exceeds 100%, but generally, when the load factor is too large, the table will be expanded by increasing the number of buckets to avoid a significant performance decline.

#### Rehashing

The core idea of <i>Rehashing</i> is that when a hash collision occurs, another hash function is used to recalculate until there is no longer a collision. Although this method is simple to implement, it requires maintaining multiple hash functions, multiple calculations, and once all hash functions cannot avoid hash collisions, this method fails.

<p>
<math display="block"><mrow><msub><mi>h</mi><mi>i</mi></msub></mrow><mrow><mi>(key)</mi></mrow> <mo>=</mo> <mrow><msub><mi>hash</mi><mi>i</mi></msub></mrow> <mrow><mi>(key)</mi></mrow> <mo>%</mo> <mrow><mi>table_size</mi></mrow> </math>
<p>

<div class="content-image">
<img src="/posts/go/structure/images/chapter_2-hash-double.png" width="60%">
<p>Insert</p>
</div>

## Go Map

### Structure

The hash table in Go language is implemented using <i>Open Hashing</i>, but its underlying implementation does not use the traditional linked list. Instead, it employs a special data structure and also introduces the concept of overflow buckets.

In Go language, a hash table is defined as `hmap`, which contains information such as the number of elements (count), hash seed (hash0), bucket pointer (buckets), and overflow bucket pointer (extra).

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

Whether it is a normal bucket or an overflow bucket, its structure is `bmap`, which contains the stored key-value pairs (keys, values), the tophash cache of the keys, the pointer to the next overflow bucket (overflow), and other information. The vast majority of the fields of `bmap` are not defined in the Go source code in the form of code. This is because the structure of `bmap` is highly optimized and its memory layout is dynamically adjusted according to the types of keys and values. It can also be seen from the structure that a `bmap` can store up to 8 key-value pairs at most. When a `bmap` is full and data is still written to it, it will be written to the `overflow` bucket.

```go
type bmap struct {
    tophash  [8]uint8
    keys     [8]keyType
    values   [8]valueType
    overflow *bmap
}
```

<div class="content-image">
<img src="/posts/go/structure/images/chapter_2-hash-go-map.png" width="80%">
<p>Go Map</p>
</div>

### Common Operation

#### Read

When accessing a `map`, single and multiple return value accesses will respectively call `runtime.mapaccess1` and `runtime.mapaccess2`. Moreover, `runtime.mapaccess2` only returns an additional boolean value indicating whether the key exists or not compared to `runtime.mapaccess1`. Additionally, Go has optimized access for some common key types; for instance, when the key is of type `int`, `runtime.mapaccess2_fast64` will be called. Taking `runtime.mapaccess2_fast64` as an example.

* If the `map` is empty or has a count of 0, return the zero value and `false` directly.

* Determine whether the `map` is being concurrently read and written. If it is, panic. This shows that the `map` is not concurrent-safe.

* If `map` has only one bucket, there is no need to calculate the hash value to locate it; simply obtain the bucket directly.

* If there is more than one bucket, calculate the hash value to determine the bucket number. If the `map` is expanding, it is necessary to determine whether the bucket has been migrated. If not, the old bucket number must be used to obtain the correct bucket.

* By traversing the tophash in the bucket, the high 8 bits of the hash value are quickly compared. If the tophash matches, the key values are further compared. If a matching key is found, the value and true are returned.

* If no matching key is found, continue to traverse the overflow buckets. If no matching key is found in all related buckets, return a zero value and false.

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

#### Write

When writing key-value pairs to a `map`, the `runtime.mapassign` function is called. The writing process is similar to the access process, but special attention should be paid to concurrent read-write checks and `map` expansion. Before writing data, it first checks if the `map` is currently expanding. If it is, it will actively call the `runtime.growWork` function to migrate data to the target bucket. After the migration is completed, the writing operation is performed in the new bucket. After writing the data, it checks the load factor and the number of overflow buckets to determine if expansion is necessary.

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

#### Delete

The logic for deleting key-value pairs is basically the same as writing, and it will also check whether the bucket is being expanded before deletion. If it is being expanded, the target bucket will be migrated first, and then the key-value pair deletion operation will be performed on the new bucket. The deletion operation will call the `runtime.mapdelete` function. For the specific implementation, please refer to the Go source code.

#### Expand

When discussing writing to a `map`, it has been observed that there are two factors that trigger expansion, one is an overly large load factor, and the other is an excessive number of overflow buckets. During expansion, the `runtime.hasGrow` function will be called.

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

The expansion operations performed by the Go language for these two situations are different.

* Same-size Expansion

When the number of overflow buckets is too large and triggers an expansion, an <i>Same-size Expansion</i> is triggered. The purpose is to organize the storage structure of the `map`, reduce the number of overflow buckets, and thereby improve access efficiency. Therefore, during an <i>Same-size Expansion</i>, the number of buckets in the new bucket group is the same as that in the old bucket group. Only by rehashing the key-value pairs to the new buckets can the use of overflow buckets be reduced.

* Double-size Expansion

When the load factor is too large and causes expansion, it will trigger a <i>Double-size Expansion</i>. At this time, the number of buckets in the new bucket group is twice that of the old bucket group. The main purpose is to reduce the load factor, decrease the probability of hash collisions, and improve access efficiency.

Moreover, the expansion of `map` is progressive. Even if an expansion operation is in progress, it will not block the read, write, and delete operations on `map`. In the section introducing the read, write, and delete operations of `map`, the handling logic for `map` during the expansion phase can also be seen.

## Summary

* Hash tables are a common and important data structure. To achieve a high-performance hash table, two aspects need to be considered: the selection of the hash function and the handling of hash collisions. The hash function should be chosen to output as uniformly distributed as possible. The main methods for resolving hash collisions include <i>Open Addressing</i>, <i>Open Hashing</i>, and <i>Rehashing</i>.

* The Go language hash table is implemented using <i>Open Hashing</i>, and it designs a special bucket structure `bmap`. Additionally, it adopts an overflow bucket strategy to reduce the frequency of hash table expansion.

* The read, write and delete operations of Go language hash tables have similar access logic. First, the hash value is calculated through the hash function and hash seed, and then the normal bucket and overflow bucket are traversed in sequence. The write operation may trigger the expansion of the hash table.

* There are two scenarios for the expansion of Go language hash tables: <i>Same-size Expansion</i> and <i>Double-size Expansion</i>. <i>Same-size Expansion</i> is triggered by an excessive number of overflow buckets. Its main purpose is to rehash key-value pairs to new buckets, organize the hash table structure, reduce the number of overflow buckets, and lower the usage frequency of overflow buckets to improve access efficiency. <i>Double-size Expansion</i> is triggered by an excessively high load factor. Its main purpose is to increase the number of buckets to reduce the load factor, decrease the occurrence of hash collisions, and thereby enhance access efficiency.