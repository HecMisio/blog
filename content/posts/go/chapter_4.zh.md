---
layout: post
title: "深入理解 Go 并发"
description: "本文将全面解析 Go 语言中的并发编程，包括并发原语、Channel 的使用、Context 的管理、计时器的使用以及 Go 独特的 GMP 模型"
date: 2025-04-03T09:31:24+08:00
image: "/posts/go/images/chapter_4-cover.jpg"
tags: ["Go"]
---

Go 语言以其强大的并发模型而闻名，这使得开发者能够轻松地编写高效、可扩展的并发程序。在这篇文章中，我们将深入探讨 Go 并发的核心概念，包括 Goroutine 的使用、Channel 的通信机制、Context 的上下文管理，以及 Go 独特的 GMP 模型（Goroutine、线程和处理器的调度模型）。通过这些内容，你将全面了解 Go 并发的设计哲学和实现细节，为编写高性能的 Go 应用程序打下坚实的基础。

## Goroutine

Goroutine 是 Go 语言中实现并发的核心原语。它是一种轻量级的线程，由 Go 运行时管理，而不是操作系统线程。相比传统线程，Goroutine 的启动和切换成本极低，因此可以在单个应用程序中轻松创建成千上万个 Goroutine。

* **使用场景**
  * 并行任务：将独立的任务分配到不同的 Goroutine 中执行，如处理多个独立的网络请求。
  * I/O操作：通过 Goroutine 处理 I/O 密集型任务，避免阻塞主线程。
  * 定时任务：结合 time 包，使用 Goroutine 实现定时任务。

```go
package main

import (
    "fmt"
    "time"
)

func sayHello() {
    fmt.Println("Hello, Goroutine!")
}

func main() {
    go sayHello() // start a new Goroutine
    fmt.Println("Main function")
    time.Sleep(time.Second) // make sure Goroutines have time to execute.
}
```

* **注意事项**
  * 数据竞争：多个 Goroutine 访问共享变量时，可能会导致数据竞争问题。可以使用 sync.Mutex 或 channel 来同步 Goroutine。
  * 资源泄露：确保 Goroutine 能够正常退出，避免因逻辑错误导致 Goroutine 持续运行而浪费资源。
  * 主 Goroutine 退出：主 Goroutine 退出时，所有子 Goroutine 都会被强制终止，因此需要使用 sync.WaitGroup 或其他机制等待子 Goroutine 完成。

### 栈

每个 Goroutine 都有自己独立的栈空间，栈默认初始大小只有 2KB，创建和销毁的代价都非常小，用于存储函数调用的局部变量、返回地址等，由运行时直接管理，从 stackpool 或堆分配。

stackpool 是运行时中用于管理 Goroutine 栈内存的专用内存池，它与常规的堆内存分配(mcache/mcentral/mheap)完全分离，其中维护着多种大小各不相同的栈块。每一个 P（Processor）都有本地 stackpool，因为每个 Goroutine 的栈空间都是独立的，所以采用了无锁设计。当一个 Goroutine 被创建时，先从 P 的本地 stackpool 获取 2KB 的栈内存，如果本地没有，则从全局 stackpool 批量获取一批栈块填充 P 的本地 stackpool，如果全局 stackpool 也没有，则从堆（mheap）分配新内存。

如果函数调用深度增加或存在大型局部变量时，可能会导致栈空间不足而发生栈扩容。默认情况下，栈扩容依旧从 stackpool 为 Goroutine 重新分配栈内存，但如果 stackpool 中不存在合适大小的空闲栈段，或需要分配非常大的栈时，会从堆直接分配。Go编译器会进行逃逸分析，尽量让对象分配在栈上，减少堆分配。由于栈扩缩容是相对昂贵的操作，涉及内存分配和拷贝，频繁的栈扩容/收缩会影响性能，应避免深度递归或超大局部变量。

* **栈扩容流程**
  * 分配新栈：从 stackpool 或堆分配新栈，通常是当前栈的2倍。
  * 栈拷贝：将旧栈内容复制到新栈，调整所有指向旧栈的指针。
  * 切换栈：将goroutine的执行上下文切换到新栈。
  * 释放旧栈：旧栈空间失去引用，被回收，默认回收至 stackpool，但如果栈空间过大，或 stackpool 已满时，会被释放到堆中。

当检测到栈空间使用率较低时（如大量函数返回后），Go运行时会自动收缩栈大小，释放多余内存。

* **栈缩容流程**
  * 触发缩容：当检测到栈使用率低于阈值(如 25%)。
  * 分配新栈：从 stackpool 或堆获取，分配更小的栈。
  * 复制内容后，旧栈放回 stackpool 或堆中。

在 Go 1.4 之前，栈分配算法使用分段栈，后来改为使用连续栈。分段栈通过维护栈段链表实现，实现复杂度高，并在循环中频繁发生栈扩缩容时，会导致严重的性能抖动（Hot Split 问题）。为了解决这些问题，改用连续栈，连续栈实现简单，在扩缩容时直接分配新的连续栈段，通过牺牲瞬时拷贝成本，换取稳定的高性能。

### Goroutine 泄露

Goroutine 泄露是指 Goroutine 启动后由于逻辑错误或设计缺陷，未能正常退出，导致其持续运行并占用系统资源。这种问题在高并发程序中尤为常见，可能会导致内存泄露、性能下降，甚至程序崩溃。

```go
// Common scenarios of Goroutine leaks

// Blocked Channel operations
func main() {
    ch := make(chan int)
    go func() {
        <-ch // The blockage here causes leakage.
    }()
    // The main Goroutine did not close ch, so the child Goroutine can never exit.
}

// Infinite loop
func main() {
    go func() {
        for {
            // Infinite loop
        }
    }()
}

// Unprocessed timeout
func main() {
    ch := make(chan int)
    go func() {
        select {
        case <-ch:
            // Handle normally
        }
        // Without timeout handling, it may lead to leakage.
    }()
}
```

为了避免出现 Goroutine 泄露，我们可以遵循一些 Goroutine 使用的最佳实践。

```go
// Using context to control the lifecycle of Goroutines
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    go func(ctx context.Context) {
        for {
            select {
            case <-ctx.Done():
                fmt.Println("Goroutine exiting")
                return
            default:
            if err := ctx.Err(); err != nil {
                return // Early exit
            }
                // Carry out the mission
            }
        }
    }(ctx)

    time.Sleep(time.Second)
    cancel() // Notify the Goroutine to exit.
}

// Make sure the Channel is closed properly.
func main() {
    ch := make(chan int)
    go func() {
        for val := range ch {
            fmt.Println(val)
        }
    }()
    ch <- 1
    close(ch) // Make sure to close the Channel.
}

// Set a timeout mechanism
func main() {
    ch := make(chan int)
    go func() {
        select {
        case <-ch:
            fmt.Println("Received data")
        case <-time.After(time.Second): // Timeout handling
            fmt.Println("Timeout")
        }
    }()
}
```

## 并发原语

Go 提供了一些强大的并发原语，用于在 Goroutine 之间进行通信和同步。这些原语是 Go 并发模型的核心，帮助开发者编写高效、安全的并发程序。

### sync.Mutex

sync.Mutex 和 sync.RWMutex 是 Go 标准库中提供的两种锁实现，用于控制对共享资源的并发访问。

sync.Mutex 是一种互斥锁，同一时间只允许一个 Goroutine 持有锁，其他试图获取锁的 Goroutine 会被阻塞。

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var mu sync.Mutex
    count := 0

    for i := 0; i < 5; i++ {
        go func() {
            mu.Lock()
            count++
            mu.Unlock()
        }()
    }
}
```

在 Go 1.9 之前 sync.Mutex 的实现是非公平锁，即 Goroutine 获取锁的顺序与 Goroutine 到达的顺序不一致，部分 Goroutine 可能长期无法获得锁，导致饥饿问题。Go 1.9 之后 sync.Mutex 被改进为可在公平锁和非公平锁之间切换。当队列中的 Goroutine 等待时间超过 1 ms 时，锁进入饥饿模式，新到达的 Goroutine 不会尝试自旋获取锁，而是直接加入等待队列，锁被释放时会直接将锁交给等待队列头部的 Goroutine，当最后一个等待 Goroutine 获得锁后，锁退出饥饿模式。非公平锁在低竞争情况下，性能更好，但在高竞争情况下，可能导致饥饿问题，响应时间不确定；公平锁虽然在低竞争情况下，性能一般，但在高竞争情况下避免了饥饿问题，响应时间更加均衡。

sync.Mutex 的底层实现依赖于操作系统的原子操作和线程调度机制。其核心结构如下：

```go
type Mutex struct {
    state int32
    sema  uint32
}
```

state 表示锁的状态，其中低三位分别表示是否被持有（mutexLocked），是否被从正常模式唤醒（mutexWoken），是否处于饥饿状态（mutexStarving），剩余位置用于表示等待锁释放的 Goroutine 数量；sema 表示信号量，用于 Goroutine 的阻塞和唤醒。

```go
func (m *Mutex) Lock() {
    if atomic.CompareAndSwapInt32(&m.state, 0, 1) {
        // The lock was successfully acquired.
        return
    }
    // Failed to acquire the lock, blocking the current Goroutine.
    runtime_Semacquire(&m.sema)
}

func (m *Mutex) Unlock() {
    if atomic.AddInt32(&m.state, -1) == 0 {
        // Return directly without waiting for the Goroutine.
        return
    }
    // Wake up one Goroutine in the waiting queue.
    runtime_Semrelease(&m.sema)
}
```

### sync.RWMutex

sync.RWMutex 是一种读写锁，同一时间允许多个读操作或一个写操作同时进行。

```go
var rwmu sync.RWMutex
var data map[string]string

func read(key string) string {
    rwmu.RLock()
    defer rwmu.RUnlock()
    return data[key]
}

func write(key, value string) {
    rwmu.Lock()
    defer rwmu.Unlock()
    data[key] = value
}
```

它的实现基于 sync.Mutex，并通过内部的状态字段和信号量实现了对读写操作的协调。

```go
type RWMutex struct {
    w           sync.Mutex  // Writer mutex
    writerSem   uint32      // Semaphore for write operation
    readerSem   uint32      // Semaphore for read operation
    readerCount int32       // The number of Goroutines currently performing read operations
    readerWait  int32       // The number of pending read operations
}
```

* **读锁实现**

  * 加读锁（RLock）
    * readerCount 减少，表示当前读操作结束。
    * 如果 readerCount 减少到 0，并且有写锁在等待（readerWait > 0），唤醒写锁。

  ```go
  func (rw *RWMutex) RLock() {
    if atomic.AddInt32(&rw.readerCount, 1) < 0 {
        // If readerCount is less than 0, it indicates that a write lock exists and the current Goroutine will be blocked.
        runtime_Semacquire(&rw.readerSem)
    }
  }
  ```

  * 释放读锁（RUnlock）
    * readerCount 减少，表示当前读操作结束。
    * 如果 readerCount 减少到 0，并且有写锁在等待（readerWait > 0），唤醒写锁。

  ```go
  func (rw *RWMutex) RUnlock() {
    if r := atomic.AddInt32(&rw.readerCount, -1); r < 0 {
        // If readerCount < 0, it indicates that there is a write lock waiting; wake up the write lock.
        if atomic.AddInt32(&rw.readerWait, -1) == 0 {
            runtime_Semrelease(&rw.writerSem)
        }
    }
  }
  ```

* **写锁实现**

  * 加写锁
    * 首先获取互斥锁 w，确保写操作的互斥性。
    * 然后检查 readerCount 是否为 0。如果 readerCount > 0，说明有读操作正在进行，写锁会阻塞，直到所有读操作完成。

  ```go
  func (rw *RWMutex) Lock() {
    rw.w.Lock() // Obtain a write lock
    r := atomic.AddInt32(&rw.readerCount, -rwmutexMaxReaders) // Prevent new read operations
    if r != 0 {
        // If there is a current read operation, wait for all read operations to complete.
        atomic.AddInt32(&rw.readerWait, r)
        runtime_Semacquire(&rw.writerSem)
    }
  }
  ```

  * 释放写锁
    * 如果有等待的读操作，唤醒所有读操作。
    * 释放互斥锁 w。

  ```go
  func (rw *RWMutex) Unlock() {
    r := atomic.AddInt32(&rw.readerCount, rwmutexMaxReaders) // Resume morning exercises.
    for i := 0; i < int(r); i++ {
        runtime_Semrelease(&rw.readerSem) // Wake up all waiting read operations.
    }
    rw.w.Unlock() // Release the write lock
  }
  ```

Go 的读写锁是一种公平读写锁。当一个 Goroutine 尝试获取写锁时，会阻塞后续的写锁获取，再阻塞后续的读锁获取，如果当前仍有 Goroutine 持有读锁，该 Goroutine 会进入休眠，等待读锁被释放；当该 Goroutine 释放写锁时，会先释放读锁，并唤醒所有等待读锁的 Goroutine，再释放写锁，这样可以保证读操作不会被连续的写操作“饿死”。因此 sync.RWMutex 在读多写少的场景下性能更好，而在读写均衡的情况下，使用 sync.Mutex 更加简单高效。

在 Go 1.18 中新增了 TryLock 方法。

```go
var mu sync.Mutex

if mu.TryLock() {
    defer mu.Unlock()
    // The lock was acquired successfully.
} else {
    // Failed to acquire the lock.
}
```

在使用锁时，需要注意不要使用嵌套锁，避免发生死锁，同时控制锁的粒度，只对共享数据上锁，以减少锁竞争，提高并发性能。

### sync.WaitGroup

sync.WaitGroup 用于等待一组 Goroutine 完成。

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup

    for i := 0; i < 3; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            fmt.Println("Goroutine", i)
        }(i)
    }

    wg.Wait()
    fmt.Println("All Goroutines finished")
}
```

其实现原理基于计数器和信号量机制，核心思想是通过计数器记录未完成的 Goroutine 数量，并在计数器归零时唤醒等待的 Goroutine。

```go
type WaitGroup struct {
    state1 [3]uint32 // Including counters and semaphores
    sema   uint32    // Semaphores are used to block and wake up Goroutines.
}
```

Add 方法用于增加或减少计数器的值，计数器归零时通过信号量唤醒阻塞的 Goroutine;Done 方法是 Add(-1) 的简化调用，用于减少计数器；Wait 方法阻塞调用的 Goroutine，直到计数器归零。

```go
func (wg *WaitGroup) Add(delta int) {
    state := atomic.AddUint32(&wg.state1[0], uint32(delta))
    if int32(state) < 0 {
        panic("sync: negative WaitGroup counter")
    }
    if delta < 0 && state == 0 {
        runtime_Semrelease(&wg.sema) // Wake up the waiting Goroutine
    }
}

func (wg *WaitGroup) Done() {
    wg.Add(-1)
}

func (wg *WaitGroup) Wait() {
    if atomic.LoadUint32(&wg.state1[0]) == 0 {
        return // If the counter is 0, return directly.
    }
    runtime_Semacquire(&wg.sema) // Block the current Goroutine and wait for the semaphore to be released.
}
```

### sync.Cond

sync.Cond 提供了一种 Goroutine 等待和通知的机制，适用于复杂的同步场景。

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    cond := sync.NewCond(&sync.Mutex{})
    ready := false

    go func() {
        cond.L.Lock()
        for !ready {
            cond.Wait()
        }
        fmt.Println("Condition met")
        cond.L.Unlock()
    }()

    cond.L.Lock()
    ready = true
    cond.Signal()
    cond.L.Unlock()
}
```

sync.Cond 的核心结构如下：

```go
type Cond struct {
    L Locker       // The associated lock, typically a sync.Mutex or sync.RWMutex.
    notify  notifyList // An internal notification list for managing waiting Goroutines.
    checker copyChecker // Prevent Cond from being wrongly copied.
}
```

Wait 方法会让当前 Goroutine 进入等待状态，直到被其他 Goroutine 唤醒。

```go
func (c *Cond) Wait() {
    t := runtime_notifyListAdd(&c.notify) // Add the current Goroutine to the notification list.
    c.L.Unlock()                         // Release the associated lock.
    runtime_notifyListWait(&c.notify, t) // Block the current Goroutine and wait to be awakened.
    c.L.Lock()                           // Reacquire the lock after being awakened.
}
```

Signal 方法会唤醒一个等待中的 Goroutine。

```go
func (c *Cond) Signal() {
    runtime_notifyListNotifyOne(&c.notify) // Wake up a Goroutine in the notification list.
}
```

Broadcast 方法会唤醒所有等待中的 Goroutine。

```go
func (c *Cond) Broadcast() {
    runtime_notifyListNotifyAll(&c.notify) // Wake up all the Goroutines in the notification list.
}
```

需要注意的是，调用 Wait、Signal 或 Broadcast 时，必须先获取关联的锁 L。同时为了防止虚假唤醒导致的问题，即线程在没有明确的通知或信号的情况下从等待状态被唤醒，通常需要在等待时使用 for 循环反复检查条件，而不是简单地依赖一次等待。

### sync.Once

sync.Once 确保某段代码只执行一次，通常用于单例模式或初始化操作。

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var once sync.Once

    for i := 0; i < 3; i++ {
        go func() {
            once.Do(func() {
                fmt.Println("This will only run once")
            })
        }()
    }
}
```

sync.Once 的实现依赖于原子操作和内存屏障，能够在多 Goroutine 环境下保证线程安全。

```go
type Once struct {
    done uint32      // Flag bit, indicating whether it has been executed or not.
    m    sync.Mutex  // Mutual exclusion lock, used to protect the execution of functions.
}
```

Do 方法用于执行指定的函数 f，并确保该函数只会被执行一次。

```go
func (o *Once) Do(f func()) {
    if atomic.LoadUint32(&o.done) == 1 { // Quick path: Check if it has been executed.
        return
    }
    o.m.Lock() // Lock to ensure that only one Goroutine is executed.
    defer o.m.Unlock()

    if o.done == 0 { // Check again to prevent other Goroutines from having already executed.
        defer atomic.StoreUint32(&o.done, 1) // Mark as executed.
        f() // Execute the objective function
    }
}
```

## Channel

Channel 是 Go 并发模型的核心，用于在 Goroutine 之间传递数据。它提供了一种类型安全、线程安全的通信机制，使得 Goroutine 之间可以通过消息传递而不是共享内存来协作。

我们可以只声明 Channel 而不初始化，此时的 Channel 将为 nil，对 nil Channel 进行发送、接收操作都会永久阻塞，并且对 nil Channel 执行 close 操作会触发 panic。在对 Channel 的操作中，除了关闭 nil Channel 会触发 panic，对已关闭的 Channel 进行 close 操作或发送操作也会触发 panic，此外对只接收 Channel 进行 close 操作也会触发 panic。

使用 make 函数创建 Channel，可以指定是否为缓冲 Channel。无缓冲 Channel 会阻塞发送、接收操作，直到另一端准备就绪。有缓冲 Channel，若缓冲区已满，则阻塞发送操作，直到有接收操作；若缓冲区为空，则阻塞接收操作，直到有发送操作。被关闭的 Channel 虽然无法在进行发送操作，但依旧可以执行接收操作，直到缓冲区数据被取完，取完后接收操作只会接收到 Channel 元素类型的零值。

Channel 的实现基于 Go 运行时的调度器，结合了队列、锁和 Goroutine 的阻塞与唤醒机制。核心结构定义如下：

```go
type hchan struct {
    qcount   uint           // The current number of elements in the queue
    dataqsiz uint           // The capacity of the queue (buffer size)
    buf      unsafe.Pointer // The pointer of the circular buffer
    elemsize uint16         // The size of each element
    closed   uint32         // Has the channel been closed

    sendx    uint           // The send index of the circular buffer
    recvx    uint           // The receiving index of the circular buffer

    recvq    waitq          // The queue of Goroutines waiting to be received
    sendq    waitq          // The queue of Goroutines waiting to be sent

    lock     mutex          // Mutual exclusion lock, protecting concurrent access to the Channel
}
```

* **应用场景**

  * 生产者-消费者模式：使用 Channel 实现生产者和消费者之间的通信。

    ```go
    package main

    import "fmt"

    func producer(ch chan int) {
        for i := 0; i < 5; i++ {
            ch <- i
        }
        close(ch)
    }

    func consumer(ch chan int) {
        for val := range ch {
            fmt.Println("Consumed:", val)
        }
    }

    func main() {
        ch := make(chan int)
        go producer(ch)
        consumer(ch)
    }
    ```

  * 工作池模式：使用多个 Goroutine 处理任务，通过 Channel 分发任务。

    ```go
    package main

    import (
        "fmt"
        "sync"
    )

    func worker(id int, ch <-chan int, wg *sync.WaitGroup) {
        defer wg.Done()
        for task := range ch {
            fmt.Printf("Worker %d processing task %d\n", id, task)
        }
    }

    func main() {
        tasks := make(chan int, 10)
        var wg sync.WaitGroup

        for i := 1; i <= 3; i++ {
            wg.Add(1)
            go worker(i, tasks, &wg)
        }

        for i := 1; i <= 10; i++ {
            tasks <- i
        }
        close(tasks)

        wg.Wait()
    }
    ```

  * 超时控制：使用 time.After 和 Channel 实现超时控制。

    ```go
    package main
    
    import (
        "fmt"
        "time"
    )
    
    func main() {
        ch := make(chan int)
    
        go func() {
            time.Sleep(2 * time.Second)
            ch <- 42
        }()
    
        select {
        case val := <-ch:
            fmt.Println("Received:", val)
        case <-time.After(1 * time.Second):
            fmt.Println("Timeout")
        }
    }
    ```

  * 限制并发：使用带缓冲的 Channel 作为信号量。

    ```go
    var sem = make(chan int, 10)

    func process(r *Request) {
        sem <- 1
        processRequest(r)
        <-sem
    }
    ```

  * Pipline：构建数据处理流水线。

    ```go
    func gen(nums ...int) <-chan int {
        out := make(chan int)
        go func() {
            for _, n := range nums {
                out <- n
            }
            close(out)
        }()
        return out
    }

    func sq(in <-chan int) <-chan int {
        out := make(chan int)
        go func() {
            for n := range in {
                out <- n * n
            }
            close(out)
        }()
        return out
    }

    nums := gen(2, 3, 4)
    squares := sq(nums)
    ```

## Context

Context 是 Go 1.7 引入标准库的接口，是 Go 中用于控制 Goroutine 生命周期的核心工具，特别适用于需要在多个 Goroutine 之间传递取消信号、超时控制或共享数据的场景。

该接口定义了4个需要实现的方法：

* Deadline：返回 context.Context 被取消的时间，即完成工作的截至时间。
* Done：返nnel 会在回一个 Channel，该 Cha当前工作完成或上下文中被取消后关闭，多次调用返回同一个 Channel。
* Err：返回 context.Context 结束的原因，它只会在 Done 方法对应的 Channel 关闭时返回非空值。
* Value：从 context.Context 中获取键对应的值，对于同一个上下文来说，多次调用 Value 并传入相同的 Key 会返回相同的结果，该方法可以用来传递特定的数据。

context 包中提供的 context.Background，context.TODO，context.WithCancel，context.WithDeadline，context.WithTimeout，context.WithValue 函数会返回实现该接口的私有结构体。

context.Background 和 context.TODO 都会创建一个 emptyCtx，这是一个不可取消、没有截止时间的上下文，但其含义不同，context.Background 是上下文的默认值，其他所有上下文都应该从它衍生出来，而 context.TODO 应该仅在不确定使用哪种上下文时使用。

```go
type emptyCtx int

func (e emptyCtx) Deadline() (time.Time, bool) { return time.Time{}, false }
func (e emptyCtx) Done() <-chan struct{}       { return nil }
func (e emptyCtx) Err() error                  { return nil }
func (e emptyCtx) Value(key interface{}) interface{} { return nil }
```

context.WithCancal 会创建一个 cancelCtx，其中包含了一个 done Channel，用于传递取消信号。

```go
type cancelCtx struct {
    Context
    done chan struct{} // The Channel used for cancellation notifications
    mu   sync.Mutex    // Mutual exclusion lock for protecting operations
    err  error         // The reason for the cancellation
}
```

context.WithTimeout 和 context.WithDeadline 都会创建 timerCtx，它是基于 cancelCtx 实现的，其中包含了一个 deadline 截止时间，和一个 timer 定时器，用于在超时时自动取消上下文。

```go
type timerCtx struct {
    cancelCtx
    deadline time.Time
    timer    *time.Timer
}
```

context.WithValue 会创建 valueCtx，其中包含一组键值对。

```go
type valueCtx struct {
    Context
    key, val interface{}
}
```

* **应用场景**

  * 控制 Goroutine 生命周期。通过 Context 的取消信号，可以优雅地终止 Goroutine，避免资源泄漏。

    ```go
    package main

    import (
        "context"
        "fmt"
        "time"
    )

    func main() {
        ctx, cancel := context.WithCancel(context.Background())
        defer cancel()

        go func(ctx context.Context) {
            for {
                select {
                case <-ctx.Done():
                    fmt.Println("Goroutine exiting")
                    return
                default:
                    fmt.Println("Working...")
                    time.Sleep(500 * time.Millisecond)
                }
            }
        }(ctx)

        time.Sleep(2 * time.Second)
        cancel()
        time.Sleep(1 * time.Second)
    }
    ```

  * 超时控制。在网络请求或数据库查询等场景中，使用 WithTimeout 或 WithDeadline 设置超时时间。

    ```go
    package main

    import (
        "context"
        "fmt"
        "time"
    )

    func main() {
        ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
        defer cancel()

        select {
        case <-time.After(3 * time.Second):
            fmt.Println("Operation completed")
        case <-ctx.Done():
            fmt.Println("Timeout:", ctx.Err())
        }
    }
    ```

  * 传递请求范围的数据。在 HTTP 请求处理中，可以使用 WithValue 在不同的中间件或处理器之间传递数据。

    ```go
    package main

    import (
        "context"
        "fmt"
    )

    func main() {
        ctx := context.WithValue(context.Background(), "userID", 12345)
        processRequest(ctx)
    }

    func processRequest(ctx context.Context) {
        userID := ctx.Value("userID")
        fmt.Println("Processing request for user:", userID)
    }
    ```

## 调度器

Go 的调度器是 Go 运行时的核心组件之一，它负责管理 Goroutine 的执行。Go 使用了一种称为 GMP 模型的调度机制，能够高效地利用多核 CPU，同时保持 Goroutine 的轻量级特性。

### GMP 模型

GMP 模型由以下三个核心组件组成：

* **G (Goroutine)**：表示 Goroutine，每个 Goroutine 包含其执行的函数、栈空间和任务状态。
* **M (Machine)**：表示操作系统线程，负责执行 Goroutine。
* **P (Processor)**：表示逻辑处理器，负责调度 Goroutine 到 M 上运行。

每个 P 维护一个本地的 Goroutine 队列，M 需要绑定一个 P 才能执行 Goroutine，G 是由 P 分配给 M 执行的。

### GMP 工作流程

1. **创建 Goroutine**：
   * 当创建一个新的 Goroutine 时，它会被放入当前 P 的本地队列中。
   * 如果本地队列已满，多余的 Goroutine 会被放入全局队列。

2. **调度 Goroutine**：
   * P 从本地队列中取出 Goroutine 分配给绑定的 M 执行。
   * 如果本地队列为空，P 会尝试从全局队列或其他 P 的队列中窃取 Goroutine。

3. **阻塞 Goroutine**：
   * 如果 Goroutine 因 I/O 或系统调用阻塞，M 会释放绑定的 P，并尝试从调度器获取新的 P，如果没有可用的 P，M 会进入休眠状态，等待调度器唤醒。如果此时有其他等待执行的 M，则将该 P 绑定到这些 M 上继续执行其他 Goroutine，如果没有 P 会返回调度器空闲 P 列表。
   * 阻塞的 Goroutine 会被挂起，等待事件完成后重新调度。

4. **抢占式调度**：
   * Go 运行时会定期检查 Goroutine 的运行时间，超过一定时间的 Goroutine 会被抢占，以确保其他 Goroutine 有机会运行。

### Goroutine 状态

在上文中已经讨论过 Goroutine 的栈空间管理，这里主要讨论一下任务状态。在 Go 的 GMP 调度模型中，Goroutine 的状态是调度器管理的核心部分。每个 Goroutine 都有一个状态，用于描述它当前的执行情况。以下是 Goroutine 的 9 种主要状态：

1. **`_Gidle`（空闲状态）**
   * Goroutine 尚未被使用，处于空闲状态。
   * 这种状态的 Goroutine 通常存储在 Goroutine 缓存池中，等待被复用。

2. **`_Grunnable`（可运行状态）**
   * Goroutine 已经准备好运行，但尚未被分配到线程（M）上执行。
   * 调度器会将这些 Goroutine 放入 P 的本地队列或全局队列中，等待调度。

3. **`_Grunning`（运行状态）**
   * Goroutine 正在某个线程（M）上运行。
   * 此时，Goroutine 已绑定到一个 P 和 M。

4. **`_Gsyscall`（系统调用状态）**
   * Goroutine 正在执行阻塞的系统调用。
   * 当 Goroutine 进入系统调用时，M 会释放绑定的 P，P 会去调度其他 Goroutine。

5. **`_Gwaiting`（等待状态）**
   * Goroutine 正在等待某个条件完成，例如等待 Channel 操作、锁或定时器。
   * 此时，Goroutine 不会被调度器调度。

6. **`_Gdead`（死亡状态）**
   * Goroutine 已经执行完毕，处于死亡状态。
   * 这种状态的 Goroutine 会被回收，栈空间可能被释放或复用。

7. **`_Gcopystack`（栈拷贝状态）**
   * Goroutine 正在进行栈扩容或缩容操作。
   * Go 的 Goroutine 栈是动态的，当栈空间不足时，会触发栈扩容。

8. **`_Gpreempted`（被抢占状态）**
   * Goroutine 被调度器抢占，暂停执行。
   * Go 1.14 引入了抢占式调度，当 Goroutine 长时间运行时，调度器会强制抢占它以确保其他 Goroutine 能够运行。

9. **`_Gscan`（扫描状态）**
   * Goroutine 正在被垃圾回收器扫描。
   * 在垃圾回收期间，调度器会扫描 Goroutine 的栈以标记活动对象。

#### 状态转换

以下是 Goroutine 状态的典型转换流程：

1. **创建 Goroutine**：
   * Goroutine 初始状态为 `_Gidle`。
   * 调用 `go` 关键字后，状态变为 `_Grunnable`，并加入调度队列。

2. **调度运行**：
   * 调度器将 `_Grunnable` 的 Goroutine 分配给线程（M），状态变为 `_Grunning`。

3. **阻塞或等待**：
   * 如果 Goroutine 等待 Channel 或锁，状态变为 `_Gwaiting`。
   * 如果 Goroutine 进入系统调用，状态变为 `_Gsyscall`。

4. **抢占或暂停**：
   * 如果 Goroutine 被抢占，状态变为 `_Gpreempted`。

5. **完成执行**：
   * Goroutine 执行完毕后，状态变为 `_Gdead`，等待回收。

#### 状态可视化

可以使用 Go 的运行时工具（如 `pprof` 或 `trace`）查看 Goroutine 的状态。例如：

```bash
go run main.go
go tool pprof http://localhost:6060/debug/pprof/goroutine
```

trace 会跟踪程序的执行过程，分析 Goroutine 调度、系统调用、垃圾回收等，适用于调试并发问题、分析调度行为。pprof 分析程序的性能瓶颈，例如 CPU 使用、内存分配等，适用于优化性能、定位资源消耗的热点。

```go
package main

import (
    "fmt"
    "os"
    "runtime/trace"
)

func main() {
    f, err := os.Create("trace.out")
    if err != nil {
        fmt.Println("Failed to create trace file:", err)
        return
    }
    defer f.Close()

    if err := trace.Start(f); err != nil {
        fmt.Println("Failed to start trace:", err)
        return
    }
    defer trace.Stop()

    // Your program logic here
    fmt.Println("Hello, trace!")
}

go tool trace trace.out
```

```go
package main

import (
    _ "net/http/pprof"
    "net/http"
)

func main() {
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()

    // Your program logic here
    select {}
}

// Obtain 30 seconds of CPU performance analysis data.
curl -o cpu.prof http://localhost:6060/debug/pprof/profile
go tool pprof cpu.prof
top: Show the most time-consuming functions.
list: View detailed information about a specific function.
web: Generate graphical performance analysis reports (Graphviz installation required).
```
