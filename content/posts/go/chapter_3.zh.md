---
layout: post
title: "Go内置函数和关键字"
description: "讨论Go语言中一些非常重要，且非常容易编程错误的内置函数和关键字"
date: 2025-03-20T09:23:38+08:00
image: "/posts/go/images/chapter_3-cover.jpg"
tags: ["Go"]
---

Go语言中提供了许多内置函数和关键字，用于简化开发过程并支持语言的核心功能，但其中有些函数和关键字在开发中需要额外关注，它们容易引发一些意料之外的错误。

## 内置函数

### new 和 make

在 Go 语言中，new 和 make 是用于内存分配的两个内置函数，但是它们的行为有所不同。

new 用于为类型分配内存，并返回该类型零值的指针，它适用于任何类型，包括引用类型、接口、函数，只不过接口指针和函数指针并没有什么用，所以很少见。

```go
type Person struct {
    Name string
    Age  int
}

// *T = new(T)
p := new(Person)
fmt.Println(p.Name)
```

```
当我们使用结构体指针去访问结构体时，通常不需要显式的使用 * ，但使用其他类型的指针时就必须使用 * ，这其实是Go提供的一种语法糖，当使用结构体指针时可以省略显式的解引用操作，编译器会在编译时自动将 p.Name 转化为 (*p).Name，不过这种语法糖只对结构体指针有效。
```

make 用于创建并初始化切片、映射和通道这三种引用类型，它返回的并不是指针，而是这些引用类型的值。

```go
// T = make(T, args)
s := make([]int, 5)

m := make(map[string]int)

ch := make(chan int, 10)
```

```
引用类型和指针类型都涉及对数据的间接访问，并且引用类型对数据的间接访问实际上就是通过指针类型实现的，但它们通常还包含了其他的一些数据，方便更好的管理和共享底层数据。
```

### len 和 cap

在Go语言中，len 和 cap 用于获取集合类型的长度和容量，但在适用性上有所不同。

<div class="table-container">
<table>
<tr><th>类型</th><th>len</th><th>cap</th></tr>
<tr><th>string</th><td>返回字节数</td><td>不适用</td></tr>
<tr><th>array</th><td>返回元素数量</td><td>返回容量（等于数组长度）</td></tr>
<tr><th>slice</th><td>返回元素数量</td><td>返回容量</td></tr>
<tr><th>map</th><td>返回键值对数量</td><td>不适用</td></tr>
<tr><th>channel</th><td>返回未读取元素数量</td><td>返回缓冲区容量</td></tr>
</table>
</div>

```
为什么 cap 可以用于数组、切片，却不能用于 string 呢，毕竟 string 的底层就是byte数组啊？

我想这是因为在Go中 string 是不可变类型，获取 string 并没有什么意义。
```

```
数组也是不可变的呀？

的确，不过数组和切片、通道都属于集合类型，所以为了统一行为就保留了cap 对数组的操作。
```

```
那为什么 cap 也不能用于映射呢？

映射的容量在运行时是动态变化的，即使在使用 make 初始化时指定 cap，也只是告知编译器预计存储的键值对数量，实际底层映射的容量并不等于这个值，因此映射的容量对于开发者来说没有什么意义。
```

### append

append 用于向切片追加元素。

如果切片的容量足够，append 会直接在底层数组中追加元素，并返回更新后的切片；如果切片的容量不足，append 会创建一个新的底层数组，将原切片的元素复制到新数组，并追加新元素，然后返回指向新数组的切片。

```go
s := make([]int, 0, 10)
s = append(s, 1, 2, 3)
```

append 不是并发安全的。如果多个 goroutine 同时操作同一个切片，可能会导致数据竞争。需要使用锁或其他同步机制来保证安全。

```go
var (
    s  []int
    mu sync.Mutex
)

func appendSafe(value int) {
    mu.Lock()
    s = append(s, value)
    mu.Unlock()
}
```

将一个切片作为参数传递给一个函数，在函数内部使用append向切片追加元素，虽然切片是引用类型，但如果append触发扩容，返回的新切片与函数外的切片将不再共享同一个底层数组。如果希望函数内外的切片始终一致，需要将函数内的切片作为返回值，并在函数外重新赋值。

```go
func appendSlice(s []int) []int {
    s = append(s, 4, 5, 6)
    return s
}

func main() {
    s := []int{1, 2, 3}
    s = appendSlice(s)
}
```

### copy

copy 用于将一个切片的内容复制到另一个切片。

```go
n := copy(dst, src)
```

copy 只会复制 dst 和 src 中长度较小的那个切片的元素数量。如果 dst 的长度小于 src，只会复制 dst 的长度数量的元素；如果 src 的长度小于 dst，只会复制 src 的长度数量的元素；**如果 dst 或 src 是 nil 切片，copy 会正常执行，但不会复制任何元素。**

```go
src := []int{1, 2, 3, 4}
dst := make([]int, 2)
n := copy(dst, src)
fmt.Println(dst) // output: [1 2]
fmt.Println(n)   // output: 2
```

另外，copy 可以正确的处理目标切片与源切片重叠的情况。

```go
s := []int{1, 2, 3, 4}
n := copy(s[1:], s[:3])
fmt.Println(s)          // output: [1 1 2 3]
fmt.Println(n)          // output: 3
```

### panic 和 recover

panic 和 recover 是 Go 语言中用于处理程序异常和恢复的机制。

panic 用于引发一个运行时错误，终止当前函数的执行，并开始执行 defer 语句。如果没有被 recover 捕获，panic 会逐层向上传播，直到程序崩溃，即使该 panic 发生在子 goroutine 中，这种行为被称为恐慌传播​（panic propagation）。

recover 用于捕获 panic，并恢复程序的正常执行。recover 只能在 defer 函数中生效，并且 recover 只能捕获当前 goroutine 中的 panic，不能捕获其他 goroutine 的 panic。

```go
func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()
    go func() {
        panic("panic in goroutine")
    }()
    time.Sleep(time.Second) // main goroutine cannot capture the panic of sub goroutine
}
```

### close

close 用于关闭通道，关闭通道可以通知接收方不再有数据发送，从而避免接收方一直阻塞等待。

但是需要注意的是，当 close 尝试关闭 nil 通道、只读通道（<-chan）和已关闭的通道时，会引发 panic。

```go
func receiver(ch <-chan int) {
    for v := range ch {
        fmt.Println("Received:", v)
    }
}
```

这些规则大部分都好理解，但为什么 close 也不能对只读通道进行关闭呢？这是因为只读或只写通道通常用于限制函数对通道的操作权限，一个以只读通道作为传参的函数只能对该通道进行读取操作，而Go认为关闭通道的行为应该是发送方的职责，所以为了避免接收方关闭通道，就禁止了 close 对只读通道进行关闭，防止接收方误用。

## 关键字
### for-range

for-range 用于遍历集合数据，包括数组、切片、映射、字符串、通道等。

当在 for 循环中使用 goroutine 时，需要注意循环变量的捕获问题。

```go
for i := 0; i < 3; i++ {
    go func() {
        fmt.Println(i) // may output: 3, 3, 3
    }()
}
time.Sleep(time.Second)

for i := 0; i < 3; i++ {
    go func(i int) {
        fmt.Println(i) // output: 0, 1, 2
    }(i)
}
time.Sleep(time.Second)
```

另外 range 循环遍历集合时，返回的是集合元素的副本，而不是原始元素。因此当 range 遍历的集合元素不是指针或引用时，想要修改底层数据，应当用索引下标访问。

```go
s := []int{1, 2, 3}
for _, v := range s {
    v = v * 2
}
fmt.Println(s) // output: [1 2 3]

s := []int{1, 2, 3}
for i := 0; i < len(s); i++ {
    s[i] = s[i] * 2
}
fmt.Println(s) // output: [2 4 6]
```

### select

select 用于多路复用通道操作的关键字，它可以同时监听多个通道的读写操作，并在其中一个通道准备好时执行对应的分支。

select 的执行具有随机性，如果有多个通道同时准备好，select 会随机选择一个执行，而不是按照代码顺序执行。监听 nil 通道的分支将被忽略，永远不会执行。

```go
ch1 := make(chan int)
var ch2 chan int

go func() {
    ch1 <- 1
}()

select {
case v := <-ch1:
    fmt.Println("Received from ch1:", v)
case v := <-ch2:
    fmt.Println("Never received")
default:
    fmt.Println("No data received")
}
```

select 通常与 for 循环结合使用，以便持续监听多个通道。

```go
ch := make(chan int)
done := make(chan struct{})

go func() {
    for i := 0; i < 3; i++ {
        ch <- i
    }
    close(done)
}()

for {
    select {
    case v := <-ch:
        fmt.Println("Received:", v)
    case <-done:
        fmt.Println("Done")
        return
    }
}
```

可以使用 time.After 实现超时控制。

```go
ch := make(chan int)

select {
case v := <-ch:
    fmt.Println("Received:", v)
case <-time.After(time.Second):
    fmt.Println("Timeout")
}
```

可以使用 context 实现更复杂的超时和取消逻辑。

```go
ctx, cancel := context.WithTimeout(context.Background(), time.Second)
defer cancel()

ch := make(chan int)

select {
case v := <-ch:
    fmt.Println("Received:", v)
case <-ctx.Done():
    fmt.Println("Context done:", ctx.Err())
}
```

### defer

defer 是 Go 语言中用于延迟执行函数调用的关键字。它通常用于确保资源释放、解锁、关闭文件等操作，无论函数是否正常返回或发生 panic。

defer 语句会将函数调用压入一个栈中，在当前函数返回时（无论是正常返回还是 panic），这些函数调用会按照 ​后进先出（LIFO）​ 的顺序执行。

```go
func main() {
    defer fmt.Println("Deferred 1")
    defer fmt.Println("Deferred 2")
    fmt.Println("Main function")
}

Main function
Deferred 2
Deferred 1
```

defer 的参数会在 defer 语句执行时立即求值，而不是在延迟函数调用时求值。但这种参数快照会在两种情况下失效，一种是使用匿名函数闭包时，还有一种是传参为引用类型时。

```go
func main() {
    x := 1
    defer fmt.Println("Deferred:", x)
    x = 2
    fmt.Println("Main function:", x)
}
Main function: 2
Deferred: 1

func main() {
    for i := 0; i < 3; i++ {
        defer func() {
            fmt.Println("Deferred in loop:", i)
        }()
    }
    fmt.Println("Main function")
}
Main function
Deferred in loop: 3
Deferred in loop: 3
Deferred in loop: 3

func main() {
    for i := 0; i < 3; i++ {
        defer func(i int) {
            fmt.Println("Deferred in loop:", i)
        }(i)
    }
    fmt.Println("Main function")
}
Main function
Deferred in loop: 2
Deferred in loop: 1
Deferred in loop: 0
```