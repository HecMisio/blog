---
layout: post
title: "Byte, Rune And String"
description: "Analyze the differences between byte and rune in the Go language, as well as their relationship with string."
date: 2025-02-13T10:58:33+08:00
image: "/posts/go/images/chapter_1-cover.jpg"
tags: ["Go"]
---
## Byte

The `byte` type in Go language is an alias of the `uint8` type, which distinguishes the byte type from the integer type and fully supports ASCII codes.

```go
func main() {
	var a byte = 'a'
	println(a) // 97
}
```

ASCII code is a computer coding system based on the Latin alphabet, using 1 byte (8 bits) for representation. The first bit of the 8 binary digits is uniformly set to 0, and the remaining 7 bits are used to represent specific characters. It can represent 128 characters, with the numbering range being 0-127. Among them, the displayable characters (A-Za-z, 0-9, English symbols) have a numbering range of 32-126, while 0-31 and 127 represent control characters.

## Rune

The `rune` type in Go language is an alias of the `int32` type, which distinguishes the character type from the integer type and fully supports Unicode encoding. Since Go uses UTF-8 encoding and supports the display of multiple languages, the design of the `rune` type can better handle multi-language texts.

```go
func main() {
	var i rune = '我'
	println(i) // 25105
}
```

UTF-8 is a coding method based on Unicode (a character encoding scheme that can accommodate all the characters and symbols in the world, capable of representing 1,114,112 characters). It uses a single byte as the coding unit, with variable length, and a character is composed of 1 to 4 bytes. Chinese characters are usually represented by 3 bytes.

* For single-byte characters, the first bit is set to 0, and the remaining 7 bits are used to represent the Unicode encoding of the character, which is the same as the ASCII code.
* For N-byte characters (N > 1), the first N bits of the first byte are set to 1, the (N + 1)th bit is set to 0, and the first two bits of the remaining N - 1 bytes are set to 10. The rest of the binary bits are used to represent the Unicode encoding of the character.

<div class="table-container">
  <table>
    <tr><th>Unicode Hexadecimal</th><th>UTF-8 Binary</th></tr>
    <tr><td>0000 0000 - 0000 007F</td><td>0xxxxxxx</td></tr>
    <tr><td>0000 0080 - 0000 07FF</td><td>110xxxxx 10xxxxxx</td></tr>
    <tr><td>0000 0800 - 0000 FFFF</td><td>1110xxxx 10xxxxxx 10xxxxxx</td></tr>
    <tr><td>0001 0000 - 0010 FFFF</td><td>11110xxx 10xxxxxx 10xxxxxx 10xxxxxx</td></tr>
  </table>
</div>

## Byte vs Rune

In conclusion, a variable of type `byte` cannot fully support Unicode encoding and can only support the overlapping part with ASCII code. Therefore, when attempting to assign a Chinese character to a `byte` variable, the compilation will fail.

```go
func main() {
	var a = 'a'
	var b byte = 'b'
	var c rune = '我'
	// var d byte = '我' the compilation will fail
	var d []byte = []byte("我")
	fmt.Println(a) // 97
	fmt.Println(b) // 98
	fmt.Println(c) // 25105
	fmt.Println(d) // [230 136 145]
}
```

## String

The `string` type in Go is an immutable string type that is implemented using a `byte` array. It does not store data itself but points to the underlying array through a pointer and records the length of the array.

```go
type stringStruct struct {
	str unsafe.Pointer
	len int
}
```

Therefore, when performing operations such as reassignment or concatenation on a `string` type variable, the original underlying array is not modified. Instead, a new array is generated, and the pointer of the `string` type variable is redirected to the new underlying array.

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

In most programming languages, the string type is designed to be immutable. This is because the string type is used extremely frequently in programming. Designing it as immutable helps ensure that it will not be accidentally modified, introducing security issues. At the same time, it also helps improve the safety and access performance of the string type in concurrent situations.

Of course, such a design also has drawbacks. If there are frequent string operations, such as concatenation, it may lead to overly frequent memory allocation and copying, affecting the performance of program execution. To address this, `strings.Builder` can be used for string concatenation to reduce the frequency of memory allocation and copying. Of course, in addition to `strings.Builder`, other methods such as `bytes.Buffer` and `strings.Join` can also be used for string concatenation.

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

## []Byte、[]Rune And String

The conversion between `[]byte`, `[]rune`, and `string` is quite common. Let's take a look at how these conversions are specifically implemented.
Firstly, it can be observed that the structure of the `string` type is very similar to that of a `slice`, with the only difference being that the `string` implementation lacks the `cap` attribute.

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

When converting from a `string` to a `[]byte`, the Go runtime allocates memory space for the `[]byte` and copies the data of the underlying `byte` array pointed to by the `str` pointer of the `string` to the newly allocated memory space. By printing the assembly code obtained after compiling the type conversion code, it can be seen that the type conversion uses the `runtime.stringtoslicebyte` function. After allocating new memory space, `runtime.stringtoslicebyte` calls the `copy` function to copy the underlying data of the `string` to the newly allocated memory space.

```go
func main() {
	var s = "hello world"
	var b = []byte(s)
	fmt.Println(b)
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
	if buf != nil && len(s) <= len(buf) {
		*buf = tmpBuf{}
		b = buf[:len(s)]
	} else {
		b = rawbyteslice(len(s))
	}
	copy(b, s)
	return b
}
```

### String -> []Rune

The process of converting a `string` to `[]rune` is similar. By printing the assembly code obtained after compiling the type conversion code, it can be seen that the type conversion uses the `runtime.stringtoslicerune` function. Unlike the conversion to `[]byte`, the length of `[]rune` is calculated through `for-range`. This is because `len(string)` obtains the length of the underlying `byte` array of the `string` type, while when using `for-range` to iterate, it is done in Unicode encoding format, and the elements obtained during the iteration are of `rune` type. Therefore, only in this way can the final length of the converted `[]rune` be calculated.

```go
func main() {
	var s = "hello world"
	var r = []rune(s)
	fmt.Println(r)
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
	for range s {
		n++
	}

	var a []rune
	if buf != nil && n <= len(buf) {
		*buf = [tmpStringBufSize]rune{}
		a = buf[:n]
	} else {
		a = rawruneslice(n)
	}

	n = 0
	for _, r := range s {
		a[n] = r
		n++
	}
	return a
}
```
As can be seen from the above, modifying the `[]byte` and `[]rune` after type conversion does not affect the original `string`.

### []Byte -> String 、[]Rune -> String

The process of converting `[]byte` and `[]rune` to `string` is similar, both involving memory allocation and copying. However, a notable difference is that since `[]byte` and `[]rune` are mutable, race detection and memory check related code are added during the conversion.

```go
// []byte -> string
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
	if n == 0 {
		// Turns out to be a relatively common case.
		// Consider that you want to parse out data between parens in "foo()bar",
		// you find the indices and convert the subslice to string.
		return ""
	}
	if raceenabled {
		racereadrangepc(unsafe.Pointer(ptr),
			uintptr(n),
			getcallerpc(),
			abi.FuncPCABIInternal(slicebytetostring))
	}
	if msanenabled {
		msanread(unsafe.Pointer(ptr), uintptr(n))
	}
	if asanenabled {
		asanread(unsafe.Pointer(ptr), uintptr(n))
	}
	if n == 1 {
		p := unsafe.Pointer(&staticuint64s[*ptr])
		if goarch.BigEndian {
			p = add(p, 7)
		}
		return unsafe.String((*byte)(p), 1)
	}
	var p unsafe.Pointer
	if buf != nil && n <= len(buf) {
		p = unsafe.Pointer(buf)
	} else {
		p = mallocgc(uintptr(n), nil, false)
	}
	memmove(p, unsafe.Pointer(ptr), uintptr(n))
	return unsafe.String((*byte)(p), n)
}

----------------------------------------------------------------------------------------------------
// []rune -> string
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
	if raceenabled && len(a) > 0 {
		racereadrangepc(unsafe.Pointer(&a[0]),
			uintptr(len(a))*unsafe.Sizeof(a[0]),
			getcallerpc(),
			abi.FuncPCABIInternal(slicerunetostring))
	}
	if msanenabled && len(a) > 0 {
		msanread(unsafe.Pointer(&a[0]), uintptr(len(a))*unsafe.Sizeof(a[0]))
	}
	if asanenabled && len(a) > 0 {
		asanread(unsafe.Pointer(&a[0]), uintptr(len(a))*unsafe.Sizeof(a[0]))
	}
	var dum [4]byte
	size1 := 0
	for _, r := range a {
		size1 += encoderune(dum[:], r)
	}
	s, b := rawstringtmp(buf, size1+3)
	size2 := 0
	for _, r := range a {
		// check for race
		if size2 >= size1 {
			break
		}
		size2 += encoderune(b[size2:], r)
	}
	return s[:size2]
}
```
Race detection and memory checking are means to help developers identify potential issues during debugging, especially in concurrent programming and memory operations. These can be enabled using compiler flags, such as `go run -race -msan -asan main.go`, which will report race conditions and memory errors during compilation.

### []Byte <-> []Rune

`[]byte` and `[]rune` do not support direct conversion but require `string` as a bridge for the conversion.

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

## Summary

* The difference between `byte` and `rune`: `byte` is an alias for `uint8`, while `rune` is an alias for `int32`. `byte` can represent ASCII codes, whereas `rune` can represent Unicode encodings.
* The `string` is immutable and is implemented using a `byte` array at the bottom layer. Operations on it will allocate memory and perform copying, generating a new underlying array.
* `[]byte` <-> `string` and `[]rune` <-> `string` can be directly converted, while `[]byte` <-> `[]rune` requires `string` as an intermediary.