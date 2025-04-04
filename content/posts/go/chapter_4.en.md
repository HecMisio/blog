---
layout: post
Title: "In-Depth Understanding of Go Concurrency"
Description: "This article will comprehensively analyze concurrent programming in the Go language, including concurrent primitives, the use of Channels, the management of Context, the use of timers, and Go's unique GMP model."
date: 2025-04-03T09:31:24+08:00
image: "/posts/go/images/chapter_4-cover.jpg"
tags: ["Go"]
---
The Go language is renowned for its powerful concurrency model, which enables developers to easily write efficient and scalable concurrent programs. In this article, we will delve into the core concepts of Go concurrency, including the use of Goroutines, the communication mechanism of Channels, the context management of Context, and Go's unique GMP model (the scheduling model of Goroutines, threads, and processors). Through these topics, you will gain a comprehensive understanding of the design philosophy and implementation details of Go concurrency, laying a solid foundation for writing high-performance Go applications.

## Goroutine

Goroutine is the core primitive for implementing concurrency in the Go language. It is a lightweight thread managed by the Go runtime rather than the operating system. Compared with traditional threads, the startup and context switching costs of Goroutines are extremely low, making it possible to easily create tens of thousands of Goroutines in a single application.

* **Usage scenarios**
  * Parallel tasks: Assign independent tasks to different Goroutines for execution, such as handling multiple independent network requests.
  * I/O operations: Handle I/O-intensive tasks through Goroutines to avoid blocking the main thread.
  * Timed tasks: Combine the time package and use Goroutines to implement timed tasks.

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

* **Notes to be Observed**
  * Data race: When multiple Goroutines access shared variables, it may lead to data race issues. You can use sync.Mutex or channels to synchronize Goroutines.
  * Resource leak: Ensure that Goroutines can exit normally to avoid wasting resources due to logical errors causing Goroutines to run continuously.
  * Main Goroutine exit: When the main Goroutine exits, all child Goroutines will be forcibly terminated. Therefore, you need to use sync.WaitGroup or other mechanisms to wait for child Goroutines to complete.

### Stack

Each Goroutine has its own independent stack space. The default initial size of the stack is only 2KB. The cost of creation and destruction is very small. It is used to store local variables of function calls, return addresses, etc., and is directly managed by the runtime, allocated from the stackpool or the heap.

Stackpool is a dedicated memory pool in the runtime for managing the stack memory of Goroutines. It is completely separated from the regular heap memory allocation (mcache/mcentral/mheap) and maintains multiple stack blocks of different sizes. Each P (Processor) has a local stackpool because the stack space of each Goroutine is independent, so a lock-free design is adopted. When a Goroutine is created, it first acquires 2KB of stack memory from the local stackpool of the P. If there is none locally, it acquires a batch of stack blocks from the global stackpool to fill the local stackpool of the P. If the global stackpool also has none, it allocates new memory from the heap (mheap).

When the depth of function calls increases or there are large local variables, it may lead to insufficient stack space and cause stack expansion. By default, stack expansion still reallocates stack memory for the Goroutine from the stackpool. However, if there is no suitable free stack segment in the stackpool or a very large stack needs to be allocated, it will be directly allocated from the heap. The Go compiler performs escape analysis to try to allocate objects on the stack as much as possible and reduce heap allocation. Since stack expansion and contraction are relatively expensive operations involving memory allocation and copying, frequent stack expansion/contraction can affect performance, and deep recursion or extremely large local variables should be avoided.

* **Stack expansion process**
  * Allocate a new stack: Allocate a new stack from the stackpool or the heap, usually twice the size of the current stack.
  * Stack copy: Copy the contents of the old stack to the new stack and adjust all pointers pointing to the old stack.
  * Switch stacks: Switch the execution context of the goroutine to the new stack.
  * Release the old stack: When the old stack loses its reference, it is reclaimed. By default, it is returned to the stackpool, but if the stack space is too large or the stackpool is full, it is released to the heap.

When the stack space usage rate is detected to be low (such as after a large number of functions return), the Go runtime will automatically shrink the stack size and release the excess memory.

* **Stack downsizing process**
  * Trigger stack shrinking: When the stack usage rate is detected to be below the threshold (such as 25%).
  * Allocate a new stack: Obtain from the stackpool or heap and allocate a smaller stack.
  * After copying the content, return the old stack to the stackpool or heap.

Before Go 1.4, the stack allocation algorithm used segmented stacks, which were later changed to continuous stacks. Segmented stacks were implemented by maintaining a linked list of stack segments, which was complex and could cause severe performance jitter (the Hot Split problem) when stack expansion and contraction occurred frequently in loops. To address these issues, continuous stacks were adopted. Continuous stacks are simpler to implement and, when expanding or contracting, directly allocate new continuous stack segments. This approach sacrifices the cost of instantaneous copying to achieve stable high performance.

### Goroutine Leak

Goroutine leakage refers to the situation where a Goroutine, after being initiated, fails to exit normally due to logical errors or design flaws, thus continuing to run and consume system resources. This issue is particularly common in high-concurrency programs and may lead to memory leaks, performance degradation, and even program crashes.

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

To avoid Goroutine leaks, we can follow some best practices for using Goroutines.

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

## Concurrent Primitives

Go provides some powerful concurrency primitives for communication and synchronization among Goroutines. These primitives are at the core of Go's concurrency model and help developers write efficient and safe concurrent programs.

### sync.Mutex

sync.Mutex and sync.RWMutex are two lock implementations provided by the Go standard library, used to control concurrent access to shared resources.

sync.Mutex is a mutual exclusion lock that only allows one Goroutine to hold the lock at a time. Other Goroutines attempting to acquire the lock will be blocked.

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

Before Go 1.9, the implementation of sync.Mutex was a non-fair lock, meaning that the order in which Goroutines acquired the lock was not consistent with the order in which they arrived. Some Goroutines might be unable to obtain the lock for a long time, leading to starvation issues. After Go 1.9, sync.Mutex was improved to be switchable between fair and non-fair locks. When the waiting time of Goroutines in the queue exceeds 1 ms, the lock enters starvation mode. New arriving Goroutines will not attempt to spin to acquire the lock but directly join the waiting queue. When the lock is released, it will be directly handed over to the Goroutine at the head of the waiting queue. After the last waiting Goroutine acquires the lock, the lock exits starvation mode. Non-fair locks perform better in low contention scenarios, but in high contention scenarios, they may cause starvation issues and have unpredictable response times. Fair locks, although generally having average performance in low contention scenarios, avoid starvation issues in high contention scenarios and have more balanced response times.

The underlying implementation of sync.Mutex relies on the atomic operations and thread scheduling mechanisms of the operating system. Its core structure is as follows:

```go
type Mutex struct {
    state int32
    sema  uint32
}
```

The state field represents the lock's status, with the lowest three bits respectively indicating whether it is held (mutexLocked), whether it has been awakened from the normal mode (mutexWoken), and whether it is in a starved state (mutexStarving). The remaining bits are used to represent the number of Goroutines waiting for the lock to be released; sema represents the semaphore, which is used for blocking and waking up Goroutines.

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

sync.RWMutex is a read-write lock that allows multiple read operations or a single write operation to occur simultaneously at the same time.

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

Its implementation is based on sync.Mutex and coordinates read and write operations through internal state fields and semaphores.

```go
type RWMutex struct {
    w           sync.Mutex  // Writer mutex
    writerSem   uint32      // Semaphore for write operation
    readerSem   uint32      // Semaphore for read operation
    readerCount int32       // The number of Goroutines currently performing read operations
    readerWait  int32       // The number of pending read operations
}
```

* **Read lock implementation**

  * Acquire read lock (RLock)
    * Decrease readerCount, indicating the current read operation has ended.
    * If readerCount decreases to 0 and there are write locks waiting (readerWait > 0), wake up the write lock.

  ```go
  func (rw *RWMutex) RLock() {
    if atomic.AddInt32(&rw.readerCount, 1) < 0 {
        // If readerCount is less than 0, it indicates that a write lock exists and the current Goroutine will be blocked.
        runtime_Semacquire(&rw.readerSem)
    }
  }
  ```

  * Release read lock (RUnlock)
    * readerCount decreases, indicating the current read operation has ended.
    * If readerCount reduces to 0 and there are write locks waiting (readerWait > 0), wake up the write lock.

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

* **Write lock implementation**

  * Acquire write lock
    * First, obtain the mutex lock w to ensure mutual exclusion of write operations.
    * Then, check if readerCount is 0. If readerCount > 0, it indicates that there are ongoing read operations. The write lock will be blocked until all read operations are completed.

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

  * Release the write lock.
    * If there are waiting read operations, wake up all read operations.
    * Release the mutex lock w.

  ```go
  func (rw *RWMutex) Unlock() {
    r := atomic.AddInt32(&rw.readerCount, rwmutexMaxReaders) // Resume morning exercises.
    for i := 0; i < int(r); i++ {
        runtime_Semrelease(&rw.readerSem) // Wake up all waiting read operations.
    }
    rw.w.Unlock() // Release the write lock
  }
  ```

The read-write lock in Go is a fair read-write lock. When a Goroutine attempts to acquire a write lock, it blocks subsequent write lock acquisitions and then subsequent read lock acquisitions. If there are still Goroutines holding read locks at this time, the Goroutine will go to sleep and wait for the read locks to be released. When the Goroutine releases the write lock, it first releases the read locks and wakes up all Goroutines waiting for read locks, and then releases the write lock. This ensures that read operations will not be starved by consecutive write operations. Therefore, sync.RWMutex performs better in scenarios with more reads and fewer writes, while in scenarios with balanced reads and writes, using sync.Mutex is simpler and more efficient.

The TryLock method was newly added in Go 1.18.

```go
var mu sync.Mutex

if mu.TryLock() {
    defer mu.Unlock()
    // The lock was acquired successfully.
} else {
    // Failed to acquire the lock.
}
```

When using locks, it is important to avoid nested locks to prevent deadlocks. At the same time, control the granularity of the locks and only lock shared data to reduce lock contention and improve concurrent performance.

### sync.WaitGroup

The sync.WaitGroup is used to wait for a group of Goroutines to complete.

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

Its implementation principle is based on the counter and semaphore mechanism. The core idea is to record the number of unfinished Goroutines through a counter and wake up the waiting Goroutines when the counter reaches zero.

```go
type WaitGroup struct {
    state1 [3]uint32 // Including counters and semaphores
    sema   uint32    // Semaphores are used to block and wake up Goroutines.
}
```

The Add method is used to increase or decrease the value of the counter. When the counter reaches zero, it wakes up the blocked Goroutine through a semaphore. The Done method is a simplified call to Add(-1) and is used to decrease the counter. The Wait method blocks the calling Goroutine until the counter reaches zero.

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

sync.Cond provides a mechanism for Goroutines to wait and notify, which is suitable for complex synchronization scenarios.

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

The core structure of sync.Cond is as follows:

```go
type Cond struct {
    L Locker       // The associated lock, typically a sync.Mutex or sync.RWMutex.
    notify  notifyList // An internal notification list for managing waiting Goroutines.
    checker copyChecker // Prevent Cond from being wrongly copied.
}
```

The Wait method will put the current Goroutine into a waiting state until it is awakened by another Goroutine.

```go
func (c *Cond) Wait() {
    t := runtime_notifyListAdd(&c.notify) // Add the current Goroutine to the notification list.
    c.L.Unlock()                         // Release the associated lock.
    runtime_notifyListWait(&c.notify, t) // Block the current Goroutine and wait to be awakened.
    c.L.Lock()                           // Reacquire the lock after being awakened.
}
```

The Signal method will wake up a waiting Goroutine.

```go
func (c *Cond) Signal() {
    runtime_notifyListNotifyOne(&c.notify) // Wake up a Goroutine in the notification list.
}
```

The Broadcast method will wake up all waiting Goroutines.

```go
func (c *Cond) Broadcast() {
    runtime_notifyListNotifyAll(&c.notify) // Wake up all the Goroutines in the notification list.
}
```

It should be noted that when invoking Wait, Signal or Broadcast, the associated lock L must be acquired first. Additionally, to prevent issues caused by spurious wakeups, where a thread is awakened from the waiting state without a clear notification or signal, it is usually necessary to use a for loop to repeatedly check the condition while waiting, rather than simply relying on a single wait.

### sync.Once

sync.Once ensures that a certain piece of code is executed only once, and is typically used in singleton patterns or initialization operations.

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

The implementation of sync.Once relies on atomic operations and memory barriers, ensuring thread safety in a multi-Goroutine environment.

```go
type Once struct {
    done uint32      // Flag bit, indicating whether it has been executed or not.
    m    sync.Mutex  // Mutual exclusion lock, used to protect the execution of functions.
}
```

The Do method is used to execute the specified function `f` and ensures that the function is executed only once.

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

Channel is the core of Go's concurrent model and is used to pass data between Goroutines. It provides a type-safe and thread-safe communication mechanism, enabling Goroutines to collaborate through message passing rather than shared memory.

We can declare a Channel without initializing it. In this case, the Channel will be nil. Performing send or receive operations on a nil Channel will cause a permanent block, and closing a nil Channel will trigger a panic. In Channel operations, apart from closing a nil Channel which will trigger a panic, closing or sending to a closed Channel will also trigger a panic. Additionally, closing a Channel that is only for receiving will trigger a panic.

Channels can be created using the `make` function, and it is possible to specify whether they are buffered or not. Unbuffered Channels will block send and receive operations until the other end is ready. For buffered Channels, if the buffer is full, the send operation will be blocked until a receive operation occurs; if the buffer is empty, the receive operation will be blocked until a send operation occurs. A closed Channel cannot perform send operations, but it can still execute receive operations until all data in the buffer is retrieved. After the buffer is empty, receive operations will only receive the zero value of the Channel's element type.

The implementation of Channel is based on the scheduler of the Go runtime, integrating queues, locks, and the blocking and waking mechanisms of Goroutines. The core structure is defined as follows:

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

* **Application scenarios**

  * Producer-Consumer Pattern: Use Channel to implement communication between producers and consumers.

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

  * Work pool pattern: Use multiple Goroutines to handle tasks and distribute tasks through Channels.

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

  * Timeout control: Implement timeout control using time.After and Channel.

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

  * Limit concurrency: Use a buffered channel as a semaphore.

    ```go
    var sem = make(chan int, 10)

    func process(r *Request) {
        sem <- 1
        processRequest(r)
        <-sem
    }
    ```

  * Pipeline: Build a data processing pipeline.

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

The Context interface was introduced in the Go 1.7 standard library and is a core tool in Go for controlling the lifecycle of Goroutines. It is particularly suitable for scenarios where cancellation signals need to be passed between multiple Goroutines, timeouts need to be controlled, or data needs to be shared.

This interface defines four methods that need to be implemented:

* Deadline: Returns the time when the context.Context is canceled, which is the deadline for completing the work.
* Done: Returns a channel that is closed when the current work is completed or the context is canceled. Calling this method multiple times will return the same channel.
* Err: Returns the reason for the context.Context ending. It only returns a non-empty value when the channel returned by the Done method is closed.
* Value: Retrieves the value associated with a key from the context.Context. For the same context, calling Value multiple times with the same key will return the same result. This method can be used to pass specific data.

The functions context.Background, context.TODO, context.WithCancel, context.WithDeadline, context.WithTimeout, and context.WithValue provided in the context package return private struct types that implement this interface.

Both context.Background and context.TODO create an emptyCtx, which is an uncancelable context with no deadline. However, they have different meanings. context.Background is the default value for contexts, and all other contexts should be derived from it. On the other hand, context.TODO should only be used when it is not clear which context to use.

```go
type emptyCtx int

func (e emptyCtx) Deadline() (time.Time, bool) { return time.Time{}, false }
func (e emptyCtx) Done() <-chan struct{}       { return nil }
func (e emptyCtx) Err() error                  { return nil }
func (e emptyCtx) Value(key interface{}) interface{} { return nil }
```

The context.WithCancel function creates a cancelCtx, which includes a done channel for transmitting cancellation signals.

```go
type cancelCtx struct {
    Context
    done chan struct{} // The Channel used for cancellation notifications
    mu   sync.Mutex    // Mutual exclusion lock for protecting operations
    err  error         // The reason for the cancellation
}
```

Both context.WithTimeout and context.WithDeadline create a timerCtx, which is implemented based on cancelCtx. It includes a deadline time and a timer that automatically cancels the context when the timeout occurs.

```go
type timerCtx struct {
    cancelCtx
    deadline time.Time
    timer    *time.Timer
}
```

The context.WithValue method creates a valueCtx that contains a set of key-value pairs.

```go
type valueCtx struct {
    Context
    key, val interface{}
}
```

* **Application scenarios**

  * Control the lifecycle of Goroutines. By using the cancellation signal of Context, Goroutines can be gracefully terminated to avoid resource leaks.

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

  * Timeout control. In scenarios such as network requests or database queries, use WithTimeout or WithDeadline to set the timeout period.

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

  * Pass the data of the request scope. In HTTP request processing, WithValue can be used to pass data between different middleware or processors.

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

## Scheduler

The scheduler of Go is one of the core components of the Go runtime, responsible for managing the execution of Goroutines. Go employs a scheduling mechanism known as the GMP model, which can efficiently utilize multi-core CPUs while maintaining the lightweight nature of Goroutines.

### GMP Model

The GMP model consists of the following three core components:

* **G (Goroutine)**: Represents a Goroutine, each of which contains the function it executes, stack space, and task status.
* **M (Machine)**: Represents an operating system thread, responsible for executing Goroutines.
* **P (Processor)**: Represents a logical processor, responsible for scheduling Goroutines to run on M.

Each P maintains a local Goroutine queue. M needs to be bound to a P to execute Goroutines. G is allocated by P for M to execute.

### GMP Work Process

1. **Creating Goroutines**:

    * When a new Goroutine is created, it is placed in the local queue of the current P.
    * If the local queue is full, the excess Goroutines are placed in the global queue.

2. **Scheduling Goroutines**:

    * P takes a Goroutine from its local queue and assigns it to the bound M for execution.
    * If the local queue is empty, P will attempt to steal a Goroutine from the global queue or the queue of another P.

3. **Blocking Goroutines**:

    * If a Goroutine blocks due to I/O or system calls, M will release the bound P and attempt to acquire a new P from the scheduler. If no P is available, M will enter a dormant state and wait for the scheduler to wake it up. If there are other waiting M at this time, this P will be bound to them to continue executing other Goroutines. If there is no P, it will return the idle P list to the scheduler.
    * The blocked Goroutine will be suspended and rescheduled after the event is completed.

4. **Preemptive Scheduling**:

    * The Go runtime periodically checks the running time of Goroutines. If a Goroutine has been running for longer than a certain period, it will be preempted to ensure that other Goroutines have the opportunity to run.

### Goroutine States

In the previous text, the stack space management of Goroutines has been discussed. Here, we mainly focus on the task status. In Go's GMP scheduling model, the status of Goroutines is the core part managed by the scheduler. Each Goroutine has a status that describes its current execution situation. The following are the 9 main states of Goroutines:

1. **`_Gidle` (Idle State)**

    * The Goroutine has not been used yet and is in an idle state.
    * Goroutines in this state are usually stored in the Goroutine cache pool, waiting to be reused.

2. **`_Grunnable` (Runnable State)**

    * The Goroutine is ready to run but has not yet been assigned to a thread (M) for execution.
    * The scheduler will place these Goroutines in the local queue of P or the global queue, waiting to be scheduled.

3. **`_Grunning` (Running State)**

    * The Goroutine is running on a certain thread (M).
    * At this point, the Goroutine is bound to a P and an M.

4. **`_Gsyscall` (System Call State)**

    * The Goroutine is executing a blocking system call.
    * When a Goroutine enters a system call, M releases the bound P, and P goes on to schedule other Goroutines.

5. **`_Gwaiting` (Waiting State)**

    * The Goroutine is waiting for a certain condition to be fulfilled, such as waiting for a Channel operation, a lock, or a timer.
    * At this time, the Goroutine will not be scheduled by the scheduler.

6. **`_Gdead` (Dead State)**

    * The Goroutine has completed execution and is in a dead state.
    * Goroutines in this state will be reclaimed, and their stack space may be released or reused.

7. **`_Gcopystack` (Stack Copying State)**

    * The Goroutine is currently performing a stack expansion or contraction operation.
    * The stack of a Go Goroutine is dynamic. When the stack space is insufficient, a stack expansion is triggered.

8. **`_Gpreempted` (Preempted State)**

    * The Goroutine is preempted by the scheduler and its execution is paused.
    * Go 1.14 introduced preemptive scheduling. When a Goroutine runs for a long time, the scheduler forcibly preempts it to ensure that other Goroutines can run.

9. **`_Gscan` (Scanning State)**

    * The Goroutine is being scanned by the garbage collector.
    * During garbage collection, the scheduler scans the Goroutine's stack to mark active objects.

#### State Transition

The following is the typical transition process of Goroutine states:

1. **Creating a Goroutine**:

    * The initial state of a Goroutine is `_Gidle`.
    * After the `go` keyword is called, its state changes to `_Grunnable` and it is added to the scheduling queue.

2. **Scheduling and Execution**:

    * The scheduler assigns the `_Grunnable` Goroutine to a thread (M), and its state changes to `_Grunning`.

3. **Blocking or Waiting**:

    * If a Goroutine waits for a Channel or a lock, its state changes to `_Gwaiting`.
    * If a Goroutine enters a system call, its state changes to `_Gsyscall`.

4. **Preemption or Suspension**:

    * If a Goroutine is preempted, its state changes to `_Gpreempted`.

5. **Completion and Execution**:

    * Once a goroutine finishes its execution, its status changes to `_Gdead` and it awaits garbage collection.

#### Status Visualization

The runtime tools of Go (such as `pprof` or `trace`) can be used to view the status of Goroutines.

Trace tracks the execution process of a program, analyzing Goroutine scheduling, system calls, garbage collection, etc., and is suitable for debugging concurrent issues and analyzing scheduling behavior. Pprof analyzes the performance bottlenecks of a program, such as CPU usage and memory allocation, and is suitable for optimizing performance and locating hotspots of resource consumption.

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
