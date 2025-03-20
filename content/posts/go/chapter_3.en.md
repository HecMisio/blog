---
layout: post
title: "Built-in functions and keywords in Go"
description: "Discuss some very important and error-prone built-in functions and keywords in the Go language."
date: 2025-03-20T09:23:38+08:00
image: "/posts/go/images/chapter_3-cover.jpg"
tags: ["Go"]
---

The Go language provides many built-in functions and keywords to simplify the development process and support the core functionality of the language. However, some of these functions and keywords require extra attention during development as they can easily cause unexpected errors.

## Built-in Functions

### New And Make

In the Go language, `new` and `make` are two built-in functions used for memory allocation, but they behave differently.

The `new` keyword is used to allocate memory for a type and return a pointer to the zero value of that type. It can be applied to any type, including reference types, interfaces, and functions. However, interface pointers and function pointers are rarely seen as they are not very useful.

```go
type Person struct {
    Name string
    Age  int
}

// *T = new(T)
p := new(Person)
fmt.Println(p.Name)
```

When we use a pointer to a struct to access the struct, we usually don't need to explicitly use *, but when usingpointers of other types, we must use *. This is actually a kind of syntactic sugar provided by Go. When using a pointer to a struct, the explicit dereferencing operation can be omitted, and the compiler will automatically convert p.Name to (*p).Name at compile time. However, this syntactic sugar is only valid for pointers to structs.

The `make` function is used to create and initialize slices, maps, and channels, which are three reference types. It does not return a pointer but the values of these reference types.

```go
// T = make(T, args)
s := make([]int, 5)

m := make(map[string]int)

ch := make(chan int, 10)
```

Both reference types and pointer types involve indirect access to data, and the indirect access of reference types to data is actually implemented through pointer types. However, they usually also contain some other data to facilitate better management and sharing of the underlying data.

### Len And Cap

In the Go language, len and cap are used to obtain the length and capacity of collection types, but they differ in their applicability.

<div class="table-container">
<table>
<tr><th>type</th><th>len</th><th>cap</th></tr>
<tr><th>string</th><td>Number of bytes returned</td><td>Not applicable</td></tr>
<tr><th>array</th><td>Return the number of elements</td><td>Return the capacity (equal to the array length)</td></tr>
<tr><th>slice</th><td>Return the number of elements</td><td>Return capacity</td></tr>
<tr><th>map</th><td>Return the number of key-value pairs</td><td>Not applicable</td></tr>
<tr><th>channel</th><td>Return the number of unread elements</td><td>Return buffer capacity</td></tr>
</table>
</div>

Why can `cap` be used for arrays and slices but not for strings? After all, strings are essentially byte arrays at the bottom. I think this is because in Go, strings are immutable types, and getting the capacity of a string doesn't make much sense.

Arrays are also immutable, right? Indeed, but arrays, slices, and channels all belong to the collection type, so in order to unify their behaviors, the cap operation is retained for arrays.

Then why can't cap be used for mapping either? The capacity of a mapping is dynamically changing during runtime. Even if cap is specified when initializing with make, it only informs the compiler of the expected number of key-value pairs to be stored. The actual capacity of the underlying mapping does not equal this value. Therefore, the capacity of a mapping has no significance for developers.

### Append

The `append` function is used to add elements to a slice.

If the capacity of the slice is sufficient, `append` will directly append the element to the underlying array and return the updated slice; if the capacity of the slice is insufficient, `append` will create a new underlying array, copy the elements of the original slice to the new array, append the new element, and then return a slice pointing to the new array.

```go
s := make([]int, 0, 10)
s = append(s, 1, 2, 3)
```

The `append` operation is not concurrent-safe. If multiple goroutines operate on the same slice simultaneously, it may lead to data races. Locks or other synchronization mechanisms should be used to ensure safety.

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

Passing a slice as a parameter to a function and using `append` to add elements to the slice within the function, although slices are reference types, if `append` triggers an expansion, the new slice returned will no longer share the same underlying array with the slice outside the function. If you want the slices inside and outside the function to always be consistent, you need to return the slice within the function as a return value and reassign it outside the function.

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

### Copy

The `copy` function is used to copy the contents of one slice to another.

```go
n := copy(dst, src)
```

The `copy` function only replicates the number of elements in the shorter of the `dst` and `src` slices. If the length of `dst` is less than that of `src`, only the number of elements in `dst` will be replicated; if the length of `src` is less than that of `dst`, only the number of elements in `src` will be replicated. **If either `dst` or `src` is a `nil` slice, `copy` will execute normally but will not replicate any elements.**

```go
src := []int{1, 2, 3, 4}
dst := make([]int, 2)
n := copy(dst, src)
fmt.Println(dst) // output: [1 2]
fmt.Println(n)   // output: 2
```

In addition, copy can correctly handle the situation where the target slice overlaps with the source slice.

```go
s := []int{1, 2, 3, 4}
n := copy(s[1:], s[:3])
fmt.Println(s)          // output: [1 1 2 3]
fmt.Println(n)          // output: 3
```

### Panic And Recover

Panic and recover are the mechanisms in the Go language used for handling program exceptions and recovery.

The `panic` function is used to trigger a runtime error, terminate the execution of the current function, and start executing the `defer` statements. If not caught by `recover`, `panic` will propagate upwards layer by layer until the program crashes. Even if the panic occurs in a child goroutine, this behavior is called panic propagation.

The `recover` function is used to catch panics and resume the normal execution of the program. `recover` only takes effect in `defer` functions, and it can only catch panics in the current goroutine, not in other goroutines.

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

### Close

The "close" operation is used to close a channel. Closing a channel can notify the receiver that no more data will be sent, thereby preventing the receiver from being blocked and waiting indefinitely.

However, it should be noted that when `close` attempts to close a `nil` channel, a read-only channel (`<-chan`), or a channel that has already been closed, it will cause a panic.

```go
func receiver(ch <-chan int) {
    for v := range ch {
        fmt.Println("Received:", v)
    }
}
```

Most of these rules are easy to understand, but why can't close be used to close a read-only channel? This is because read-only or write-only channels are typically used to restrict the operation permissions of a channel for a function. A function that takes a read-only channel as a parameter can only perform read operations on that channel. Go considers closing a channel to be the responsibility of the sender. Therefore, to prevent the receiver from closing the channel, close is prohibited from being used on read-only channels to avoid misuse by the receiver.

## Keywords
### For-Range

The `for-range` loop is used to iterate over collection data, including arrays, slices, maps, strings, channels, and so on.

When using goroutines in a for loop, it is necessary to pay attention to the issue of capturing loop variables.

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

In addition, when a range loops through a collection, it returns a copy of the collection's elements rather than the original elements. Therefore, when the elements of the collection looped through by range are not pointers or references, if you want to modify the underlying data, you should access it using an index.

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

### Select

The `select` keyword is used for multiplexing channel operations. It can simultaneously monitor the read and write operations of multiple channels and execute the corresponding branch when one of the channels is ready.

The execution of `select` is random. If multiple channels are ready at the same time, `select` will randomly choose one to execute, rather than following the order in the code. The branch that listens on a `nil` channel will be ignored and will never be executed.

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

The `select` statement is usually combined with a `for` loop to continuously monitor multiple channels.

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

Timeout control can be achieved by using `time.After`.

```go
ch := make(chan int)

select {
case v := <-ch:
    fmt.Println("Received:", v)
case <-time.After(time.Second):
    fmt.Println("Timeout")
}
```

More complex timeout and cancellation logic can be implemented using context.

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

### Defer

The keyword `defer` in Go is used to delay the execution of function calls. It is typically employed to ensure that operations such as resource release, unlocking, and file closure are carried out, regardless of whether the function returns normally or encounters a panic.

The defer statement pushes a function call onto a stack. When the current function returns (whether normally or due to a panic), these function calls are executed in a last-in-first-out (LIFO) order.

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

The parameters of `defer` are evaluated immediately when the `defer` statement is executed, rather than when the deferred function is called. However, this snapshot of parameters becomes invalid in two cases: when using anonymous function closures and when passing reference types as parameters.

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