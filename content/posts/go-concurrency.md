---
title: 'Which concurrency tool should you reach for in Go?'
date: "2025-10-27"
author: "Kenan Jasim"
tags: ["golang", "concurrency"]
readTime: true
toc: true
summary: "A look at when I reach for channels, mutexes or atomics in Go, and why they are not interchangeable."
description: "A look at when I reach for channels, mutexes or atomics in Go, and why they are not interchangeable."
---

A while ago I added a goroutine to a piece of code to make it faster, and I started getting a different result each time I ran it. Not on every run, maybe one run in five, which made it harder to track down. It was a race condition, and it is a reasonable illustration of what makes concurrency awkward: the difficult part is not doing several things at once, it is doing several things at once and still being sure of the result.

Go gives you a few tools for this, and most of the work is picking the right one. This is roughly the order I reach for them.

## Concurrency is not the same as parallelism

It is worth separating the two. Parallelism is running things at the same time. Concurrency is structuring a program as independent pieces that *could* run at the same time. One is about execution and the other is about structure. If you get the structure right, so that the independent pieces do not share memory, then giving the program more cores tends to speed it up without changing the result.

The idea underneath this is Communicating Sequential Processes: write each piece as ordinary sequential code, and have the pieces communicate through channels rather than sharing state. If nothing is shared, there is nothing to fight over.

## Channels are usually the default

A channel passes a value from one goroutine to another. One side puts a value in, the other takes it out.

```go
jobs := make(chan int)

go func() {
    for i := 1; i <= 3; i++ {
        jobs <- i
    }
    close(jobs) // signal "no more values"
}()

for j := range jobs {
    fmt.Println("got", j)
}
```

The send `jobs <- i` blocks until something is ready to receive, and the `range` blocks until there is something to take. They meet, hand over one value, and carry on. Neither goroutine touches the other's memory, so there is nothing to corrupt. The `close` is how the `range` loop knows when to stop. One thing to watch is that sending on a closed channel panics, so you should only ever close a channel from the sending side.

## select for more than one channel

When you have more than one channel, `select` handles whichever is ready. The part I find most useful is that it lets you avoid waiting forever:

```go
select {
case j := <-jobs:
    fmt.Println("got", j)
case <-time.After(2 * time.Second):
    fmt.Println("gave up waiting")
}
```

If a job arrives within two seconds you handle it, and if it does not, `time.After` fires and you move on instead of blocking. I have used that pattern to deal with a lot of "why is this stuck" situations.

## When channels do not fit: mutexes

Channels work well until you have genuinely shared state, like a cache or a counter that a lot of goroutines all need to update. You can model that with channels, but you tend to end up with one goroutine "owning" the state while everything else queues to talk to it, which is really just a lock with extra steps. In that case it is simpler to use an actual lock:

```go
type Counter struct {
    mu sync.Mutex
    n  int
}

func (c *Counter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.n++
}
```

`Lock` takes ownership and makes everything else wait, and the deferred `Unlock` always releases it, including if the function panics. It is worth remembering that a mutex is effectively a queue, so the longer you hold it, the longer everything behind it waits.

## Atomics, when even a mutex is too much

For a single integer that is updated often enough that a mutex becomes a problem, there is `sync/atomic`. These operations map to single CPU instructions, so there is no lock involved:

```go
var n int64
atomic.AddInt64(&n, 1)          // safe increment, no lock
current := atomic.LoadInt64(&n) // safe read
```

I would treat this as a last resort rather than a default. It only works on integer types, and while you can build more complicated things out of `atomic.CompareAndSwap`, it gets difficult quickly. If you are reaching for atomics, it is worth measuring first, because a lot of "lock-free" code has a mutex underneath it anyway.

## Conclusion

When I am deciding which of these to use, I roughly work down a list:

1. Reach for channels first, and use `select` to coordinate them, so that you avoid shared state where you can.
2. When you do have genuinely shared state, such as a cache, a registry or a hot counter, use a mutex.
3. Only drop to atomics for a single hot integer, and only once you have measured a reason to.

Most of the time you do not need to go very far down that list.
