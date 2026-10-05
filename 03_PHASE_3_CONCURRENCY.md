# Phase 3 — Concurrency and Parallelism

## 1. Concurrency vs parallelism

Concurrency:
> multiple tasks are in progress and can make progress independently.

Parallelism:
> multiple tasks execute at the same time on multiple CPU cores.

Go is designed to make concurrency easy with goroutines and channels.

---

## 2. Goroutines

```go
go doWork()
```

A goroutine is a lightweight concurrent execution unit managed by the Go runtime.

It is much cheaper than creating a traditional OS thread for every small task.

But goroutines are not free:
- they consume memory
- they require scheduling
- they can leak
- uncontrolled creation can overload downstream systems

---

## 3. Channels

Channels communicate between goroutines.

```go
ch := make(chan int)

go func() {
    ch <- 10
}()

value := <-ch
```

A channel can be thought of as a typed communication path.

---

## 4. Buffered channels

```go
ch := make(chan int, 10)
```

The buffer allows sends without requiring an immediate receiver until the buffer is full.

Unbuffered channels provide direct synchronization between sender and receiver.

---

## 5. Channel ownership and closing

A common design rule:

> The sender that knows no more values will be sent should generally close the channel.

Receiving:

```go
for value := range ch {
    process(value)
}
```

Receiving from a closed channel returns zero values after buffered values are exhausted.

Sending on a closed channel panics.

Closing a channel twice panics.

---

## 6. select

`select` waits on multiple channel operations.

```go
select {
case v := <-ch:
    process(v)
case <-ctx.Done():
    return ctx.Err()
}
```

This is fundamental for cancellation and timeout-aware concurrency.

---

## 7. Worker pools

A worker pool controls concurrent processing.

Architecture:

```text
             jobs
               |
       +-------+-------+
       |       |       |
    worker  worker  worker
       |       |       |
       +-------+-------+
               |
            results
```

Example:

```go
jobs := make(chan Job)
results := make(chan Result)

for i := 0; i < workers; i++ {
    go worker(jobs, results)
}
```

### Why use a worker pool?

Without one:

```text
1 million jobs
→ 1 million goroutines
→ memory pressure
→ scheduling overhead
→ downstream overload
```

With a pool:

```text
1 million jobs
→ bounded workers
→ controlled concurrency
→ backpressure
```

The pool protects the system and downstream dependencies.

### Key interview point

Worker pools are not mainly about making code "more concurrent".

They are about **bounded concurrency and resource control**.

---

## 8. Go scheduler: G-M-P

The runtime scheduler is commonly explained using:

- G = goroutine
- M = machine/OS thread
- P = processor/runtime scheduling resource

Conceptually:

```text
P → schedules G onto M
M → executes G
```

`GOMAXPROCS` influences how many Ps can execute Go code simultaneously.

You generally do not manually manage this scheduler.

---

## 9. Mutex vs channel

Use a mutex when:
- you have shared mutable state
- the critical section is simple
- direct locking is clearer

Use channels when:
- goroutines need to communicate
- ownership can be transferred
- a pipeline/worker architecture fits naturally

Do not force every problem into channels.

---

## 10. Atomic operations

`sync/atomic` provides low-level atomic operations.

Useful for counters and small shared-state operations.

Example:

```go
atomic.AddInt64(&counter, 1)
```

Atomic operations can be cheaper than locks for suitable workloads, but they can make complex state harder to reason about.

---

## 11. Context

`context.Context` carries request-scoped:
- cancellation
- deadlines
- timeouts
- limited request metadata

Typical API:

```go
func GetUser(ctx context.Context, id string) error
```

---

## 12. Why context is normally the first parameter

Idiomatic Go places context first:

```go
func DoWork(ctx context.Context, input Input) error
```

Reasons:
- it is request/control-flow metadata
- callers can immediately see cancellation/deadline propagation
- it is consistent across standard libraries and APIs
- it avoids hiding cancellation inside business parameters

Context should normally be passed explicitly, not stored globally.

Do not put arbitrary application state into context.

---

## 13. Context vs defer

These solve completely different problems.

### context

Controls a running operation:

```text
cancel
timeout
deadline
request scope
```

### defer

Controls cleanup when the current function exits:

```text
Close file
Unlock mutex
Rollback/cleanup
```

A useful mental model:

```text
context = "Should this work continue?"
defer   = "What must happen when this function leaves?"
```

They often appear together.

---

## 14. Cancellation propagation

```go
ctx, cancel := context.WithTimeout(parent, time.Second)
defer cancel()
```

Downstream code should respect:

```go
select {
case <-ctx.Done():
    return ctx.Err()
case result := <-resultCh:
    return result
}
```

This prevents work from continuing after the request is gone.

---

## 15. Concurrency leaks

A goroutine leak occurs when a goroutine remains blocked or alive longer than intended.

Common causes:
- sending to a channel nobody receives from
- receiving from a channel nobody closes/writes to
- forgetting cancellation
- waiting forever on a dependency

Every goroutine should have a clear lifecycle.

---

## 16. Deadlocks

Deadlock occurs when goroutines wait forever for each other.

Typical causes:
- incorrect channel ownership
- lock ordering problems
- waiting for a result while preventing the producer from running

Think in terms of ownership and lifecycle.

---

## 17. Graceful shutdown

Production services should:
1. stop accepting new work
2. cancel/propagate shutdown
3. allow in-flight work to finish within a deadline
4. close resources
5. exit

Context is central to this pattern.
