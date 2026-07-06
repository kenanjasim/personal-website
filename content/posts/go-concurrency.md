---
title: 'Which concurrency tool should you reach for in Go?'
date: "2025-10-27"
author: "Kenan Jasim"
tags: ["golang", "concurrency"]
readTime: true
toc: true
summary: "Channels, mutexes and atomics all handle concurrency, but they're not interchangeable. Here's how I decide."
description: "Channels, mutexes and atomics all handle concurrency, but they're not interchangeable. Here's how I decide."
---

The first time I added a goroutine to make something faster, I ran it and got a different answer. Ran it again, different again. Not broken every time, which would be easy to chase, but wrong maybe one run in five. That's a race condition, and it taught me the real lesson of concurrency: the hard part isn't doing things at once, it's doing them at once and still being sure what comes out.

Go gives you a few tools for this, and picking the right one is most of the battle. Here's the order I reach for them.

## First, concurrency isn't parallelism

Parallelism is running things at the same time. Concurrency is designing your program as independent pieces that *could* run at the same time. One is execution, the other is structure. Get the structure right, independent pieces that don't share memory, and handing it more cores becomes a free speedup that never changes the result.

The old idea underneath this is Communicating Sequential Processes: write each piece as boring sequential code, and have the pieces talk through channels instead of sharing state. No shared state, nothing to fight over.

## Channels: the default

A channel is a bucket chain. A sender drops a value in, a receiver takes it out.

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

The send `jobs <- i` blocks until someone is ready to receive, and the `range` blocks until there's something to take. They meet, hand off one value, and move on. Neither goroutine ever touches the other's memory, so there's nothing to corrupt. The `close` is what lets the `range` loop know when to stop. One trap: sending to a closed channel panics, so only ever close from the sending side.

## select: routing and timeouts

When you have more than one channel, `select` picks whichever is ready. Its most useful trick is not waiting forever:

```go
select {
case j := <-jobs:
    fmt.Println("got", j)
case <-time.After(2 * time.Second):
    fmt.Println("gave up waiting")
}
```

If a job arrives within two seconds you handle it. If not, `time.After` fires and you move on instead of hanging. That one pattern has saved me from more "why is this stuck" mysteries than anything else.

## When channels don't fit: mutexes

Channels are great until you have genuinely shared state, like a cache or a counter that a hundred goroutines all need to bump. Model that with channels and you end up with one goroutine "owning" it while everyone queues to ask it questions, which is just a slow lock in disguise. So use an actual lock:

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

`Lock` says "mine now, everyone wait," and the deferred `Unlock` always releases it, even on a panic. Simple and correct. Just remember a mutex is a queue: the longer you hold it, the longer the line behind you.

## Going lower: atomics

For a single integer that's hot enough that even a mutex hurts, there's `sync/atomic`. These map to single CPU instructions, so there's no lock to hold:

```go
var n int64
atomic.AddInt64(&n, 1)          // safe increment, no lock
current := atomic.LoadInt64(&n) // safe read
```

This is a scalpel, not a default. It only works on integer types, and yes, you can build a lock out of `atomic.CompareAndSwap` if you enjoy pain. But if you're reaching here, measure first. Most "lock-free" code is a mutex underneath anyway.

## Conclusion

When I'm deciding, it's basically a ladder:

1. Reach for channels first, and use `select` to manage them. They let you avoid shared state entirely.
2. When you hit genuinely shared state (a cache, a registry, a hot counter), use a mutex.
3. Drop to atomics only for a single hot integer, and only once you've measured a reason to.

Start at the top. You almost never need to go far down.
