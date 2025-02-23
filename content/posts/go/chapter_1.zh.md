---
layout: post
title: "Byte、Rune 和 String"
description: "分析Go语言中byte、rune之间的区别，以及与string之间的关系"
date: 2025-02-13T10:58:33+08:00
image: "/posts/go/images/chapter_1-cover.png"
tags: ["Go"]
---
## Byte

`byte`类型在Go语言中是`uint8`类型的别名，以此将字节类型与整数类型区分，可以完全支持ASCII码。

```go
func main() {
	var a byte = 'a'
	println(a) // 97
}
```

ASCII码是基于拉丁字母的一套计算机编码系统，使用1个字节（8 bit）来表示，其中8个二进制位中的首位统一为0，使用后7位表示具体字符，可以表示128个字符，编号范围为0-127，其中显示字符（A-Za-z、0-9、英文符号）编号范围是32-126，0-31和127表示控制字符。

## Rune

`rune`类型在Go语言中是int32类型的别名，以此将字符类型与整数类型区分，可以完全支持Unicode编码。由于Go采用UTF-8编码，支持多语言的显示，设计`rune`类型可以更好地处理多语言文本。

```go
func main() {
	var i rune = '我'
	println(i) // 25105
}
```

UTF-8是基于Unicode（可以容纳世界上所有文字和符号的字符编码方案，共可以表示1114112个字符）的一种编码方式，采用单字节作为编码单位，长度可变，由1-4字节组成一个字符，中文通常使用3个字节表示。

* 对于单字节字符，首位设为0，后7位用于表示字符的Unicode编码，与ASCII码相同；
* 对于N字节字符（N>1），第一个字节的前N位设置为1，第N+1位设置为0，剩余的N-1个字节的前两位都设为10，其余二进制位用于表示字符的Unicode编码。

<div class="table-container">
  <table>
    <tr><th>Unicode 十六进制</th><th>UTF-8 二进制</th></tr>
    <tr><td>0000 0000 - 0000 007F</td><td>0xxxxxxx</td></tr>
    <tr><td>0000 0080 - 0000 07FF</td><td>110xxxxx 10xxxxxx</td></tr>
    <tr><td>0000 0800 - 0000 FFFF</td><td>1110xxxx 10xxxxxx 10xxxxxx</td></tr>
    <tr><td>0001 0000 - 0010 FFFF</td><td>11110xxx 10xxxxxx 10xxxxxx 10xxxxxx</td></tr>
  </table>
</div>

## Byte vs Rune

综上所述，一个`byte`类型的变量无法完全支持Unicode编码，仅能支持与ASCII码重叠的部分，因此当尝试将一个中文字符赋值给一个`byte`变量时，无法通过编译。

```go
func main() {
	var a = 'a'
	var b byte = 'b'
	var c rune = '我'
	// var d byte = '我' 无法通过编译
	var d []byte = []byte("我")
	fmt.Println(a) // 97
	fmt.Println(b) // 98
	fmt.Println(c) // 25105
	fmt.Println(d) // [230 136 145]
}
```

## String

`string`是Go语言中的字符串类型，具有不可变性，利用`byte`数组来实现，本身并不存储数据，而是通过指针指向底层数组，并记录数组长度。

```go
type stringStruct struct {
	str unsafe.Pointer
	len int
}
```

因此，当对`string`类型变量进行重新赋值、拼接等操作时，并不会修改原来的底层数组，而是重新生成新的数组，并将`string`类型变量的指针指向新的底层数组。

```go
func main() {
	var s = "你好，世界！"
        // stringStruct.str
	str := *(*unsafe.Pointer)(unsafe.Pointer(&s))
        // stringStruct.len                                       
	length := *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + unsafe.Sizeof(str)))
	fmt.Printf("%p\n", str) // 0x87d528                                                 
	fmt.Printf("%d\n", length) // 18                                                          

	s = "hello, world!"
        // stringStruct.str
	str = *(*unsafe.Pointer)(unsafe.Pointer(&s))
        // stringStruct.len                                       
	length = *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + unsafe.Sizeof(str)))
	fmt.Printf("%p\n", str) // 0x87c36d                                
	fmt.Printf("%d\n", length) // 13                                                     
}
```

在大多数编程语言中，字符串类型都是被设计成不可变的，这是因为字符串类型在编程中使用频率极高，设计成不可变的，有利于保障其不会被意外修改，引入安全问题，同时也有利于提高字符串类型在并发情况下的安全性和访问性能。

当然这样的设计也有弊端，如果出现频繁的字符串操作，如拼接，可能会导致过度频繁的内存分配和复制，影响程序执行性能。对此可以使用`strings.Builder`来进行字符串拼接，以减少内存分配和复制频率。当然，除了`strings.Builder`可以拼接字符串之外，还可以使用`bytes.Buffer`、`strings.Join`等方法进行字符串拼接。

```go
var builder strings.Builder
builder.WriteString("hello")
builder.WriteString(",world")
fmt.Println(builder.String())

var buf = new(bytes.Buffer)
buf.WriteString("hello")
buf.WriteString(",world!")
fmt.Println(buf.String())
```

## []Byte、[]Rune和String

`[]byte`、`[]rune`、`string`之间的转换比较常见，接下来来看一下它们之间转换具体是怎么实现的。
首先，可以看到`string`类型的结构与`slice`很相似，仅仅比`slice`的实现少了`cap`属性。

```go
type stringStruct struct {
	str unsafe.Pointer
	len int
}

type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```
### String -> []Byte

由`string`转`[]byte`时，Go运行时会为`[]byte`分配内存空间，并将`string`的`str`指针指向的底层`byte`数组的数据复制到新分配的内存空间上。通过打印类型转换代码经编译后得到的汇编代码，可以看到类型转换使用的是`runtime.stringtoslicebyte`函数，`runtime.stringtoslicebyte`在经过分配新的内存空间后，调用`copy`函数，将`string`的底层数据拷贝给了新分配的内存空间。

```go
func main() {
	var s = "hello world"
	var b = []byte(s)
	fmt.Println(b) // [104 101 108 108 111 32 119 111 114 108 100]
}

...
CALL    runtime.stringtoslicebyte(SB)
MOVUPS  X15, main..autotmp_10+40(SP)
PCDATA  $1, $1
CALL    runtime.convTslice(SB)
LEAQ    type:[]uint8(SB), DX
...

func stringtoslicebyte(buf *tmpBuf, s string) []byte {
	var b []byte
	if buf != nil && len(s) <= len(buf) { // 判断临时缓冲区是否足够大
		*buf = tmpBuf{}
		b = buf[:len(s)]
	} else { // 临时缓冲区不够大，则重新分配内存
		b = rawbyteslice(len(s))
	}
	copy(b, s)
	return b
}
```

### String -> []Rune

`string`转`[]rune`的过程相似，通过打印类型转换代码经编译后得到的汇编代码，可以看到类型转换使用的是`runtime.stringtoslicerune`函数，与转换成`[]byte`时不同的是，`[]rune`的长度是通过`for-range`计算出来的，这是因为`len(string)`获得的长度是`string`类型底层`byte`数组的长度，而使用`for-range`遍历时，是以Unicode编码的格式进行遍历的，遍历获得的元素是`rune`类型，因此只有这样才能计算出最终转换的`[]rune`的长度。

```go
func main() {
	var s = "hello world"
	var r = []rune(s)
	fmt.Println(r) // [104 101 108 108 111 32 119 111 114 108 100]
}

...
CALL    runtime.stringtoslicerune(SB)
MOVUPS  X15, main..autotmp_10+40(SP)
PCDATA  $1, $1
CALL    runtime.convTslice(SB)
LEAQ    type:[]int32(SB), DX
...

func stringtoslicerune(buf *[tmpStringBufSize]rune, s string) []rune {
	// two passes.
	// unlike slicerunetostring, no race because strings are immutable.
	n := 0
	for range s { // 计算[]rune的长度
		n++
	}

	var a []rune
	if buf != nil && n <= len(buf) { // 判断临时缓冲区是否足够大
		*buf = [tmpStringBufSize]rune{}
		a = buf[:n]
	} else { // 临时缓冲区不够大，则重新分配内存
		a = rawruneslice(n)
	}

	n = 0
	for _, r := range s { // 内存复制，将字符串数据复制给[]rune
		a[n] = r
		n++
	}
	return a
}
```
由上可知，修改类型转换后的`[]byte`和`[]rune`，并不会对原`string`产生影响。

### []Byte -> String 、[]Rune -> String

`[]byte`、`[]rune`转`string`的过程相似，都会涉及内存分配和复制，但有点不同的是，由于`[]byte`和`[]rune`是可变的，所以在转换时，加入了竞态检测和内存检查的相关代码。

```go
// []byte 转 string
func main() {
	var r = []byte{'h', 'e', 'l', 'l', 'o', ',', 'w', 'o', 'r', 'l', 'd', '!'}
	var s = string(r)
	fmt.Println(s)
}

...
CALL    runtime.slicebytetostring(SB)
MOVUPS  X15, main..autotmp_12+56(SP)
PCDATA  $1, $1
NOP
CALL    runtime.convTstring(SB)
LEAQ    type:string(SB), DX
...

func slicebytetostring(buf *tmpBuf, ptr *byte, n int) string {
	if n == 0 { // 处理零字节
		// Turns out to be a relatively common case.
		// Consider that you want to parse out data between parens in "foo()bar",
		// you find the indices and convert the subslice to string.
		return ""
	}
	if raceenabled { // 竞态检测
		racereadrangepc(unsafe.Pointer(ptr),
			uintptr(n),
			getcallerpc(),
			abi.FuncPCABIInternal(slicebytetostring))
	}
	if msanenabled { // 内存检查
		msanread(unsafe.Pointer(ptr), uintptr(n))
	}
	if asanenabled { // 地址清理
		asanread(unsafe.Pointer(ptr), uintptr(n))
	}
	if n == 1 { // 处理单字节
		p := unsafe.Pointer(&staticuint64s[*ptr])
		if goarch.BigEndian {
			p = add(p, 7)
		}
		return unsafe.String((*byte)(p), 1)
	}
	var p unsafe.Pointer
	if buf != nil && n <= len(buf) { // 内存分配
		p = unsafe.Pointer(buf)
	} else {
		p = mallocgc(uintptr(n), nil, false)
	}
	memmove(p, unsafe.Pointer(ptr), uintptr(n)) // 内存复制
	return unsafe.String((*byte)(p), n)
}

----------------------------------------------------------------------------------------------------
// []rune 转 string
func main() {
	var r = []rune{'h', 'e', 'l', 'l', 'o', ',', 'w', 'o', 'r', 'l', 'd', '!'}
	var s = string(r)
	fmt.Println(s)
}

...
CALL    runtime.slicerunetostring(SB)
MOVUPS  X15, main..autotmp_12+88(SP)
PCDATA  $1, $1
CALL    runtime.convTstring(SB)
LEAQ    type:string(SB), DX
...

func slicerunetostring(buf *tmpBuf, a []rune) string {
	if raceenabled && len(a) > 0 { // 竞态检测
		racereadrangepc(unsafe.Pointer(&a[0]),
			uintptr(len(a))*unsafe.Sizeof(a[0]),
			getcallerpc(),
			abi.FuncPCABIInternal(slicerunetostring))
	}
	if msanenabled && len(a) > 0 { // 内存检查
		msanread(unsafe.Pointer(&a[0]), uintptr(len(a))*unsafe.Sizeof(a[0]))
	}
	if asanenabled && len(a) > 0 { // 地址清理
		asanread(unsafe.Pointer(&a[0]), uintptr(len(a))*unsafe.Sizeof(a[0]))
	}
	var dum [4]byte
	size1 := 0
	for _, r := range a {
		size1 += encoderune(dum[:], r) // 将Unicode编码成UTF-8
	}
	s, b := rawstringtmp(buf, size1+3)
	size2 := 0
	for _, r := range a {
		// check for race
		if size2 >= size1 {
			break
		}
		size2 += encoderune(b[size2:], r) // 将Unicode编码成UTF-8
	}
	return s[:size2]
}
```
竞态检测和内存检查是为了帮助开发者在调试时发现潜在问题的手段，特别是在并发编程和内存操作中，可以使用编译标志来启用这些检测，如`go run -race -msan -asan main.go`，这样就会在编译时报告竞争情况和内存错误。

### []Byte <-> []Rune

`[]byte`和`[]rune`并不支持直接转换，而是需要借助`string`作为桥梁进行转换。

```go
func main() {
	var b1 = []byte{'h', 'e', 'l', 'l', 'o', ',', 'w', 'o', 'r', 'l', 'd', '!'}
	var r1 = []rune(string(b1))
	fmt.Println(r1)

	var r2 = []rune{'h', 'e', 'l', 'l', 'o', ',', 'w', 'o', 'r', 'l', 'd', '!'}
	var b2 = []byte(string(r2))
	fmt.Println(b2)
}
```

## 总结

* `byte`和`rune`的区别：`byte`是`uint8`的别名，而`rune`是`int32`的别名，`byte`可以表示ASCII码，而`rune`则可以表示Unicode编码。
* `string`是不可变的，其底层使用`byte`数组实现，对其操作会进行内存分配和复制，产生新的底层数组。
* `[]byte`<->`string`，`[]rune`<->`string`可以直接转换，而`[]byte`<->`[]rune`需要使用`string`作为桥梁。