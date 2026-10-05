# Phase 8 — Google / FAANG Go Interview Revision

## 1. Fundamentals you must explain cleanly

Be able to explain:

- static typing
- compilation
- package vs function
- `package main`
- `func main`
- standard library
- exported vs unexported identifiers
- variables and zero values
- arrays vs slices
- maps
- structs
- pointers
- methods
- interfaces
- nil interface trap
- errors
- defer
- panic/recover
- modules

---

## 2. Internals

Be able to explain:

- stack vs heap
- escape analysis
- garbage collection
- allocation
- slice backing arrays
- map behavior
- value vs reference-like semantics
- race conditions
- mutex/RWMutex
- atomic operations

A strong answer should explain the trade-off, not merely define the term.

---

## 3. Concurrency

Must know deeply:

- goroutines
- channels
- buffered/unbuffered channels
- select
- channel closing
- worker pools
- bounded concurrency
- backpressure
- scheduler
- G-M-P
- mutex
- atomic
- context
- cancellation
- deadlines
- goroutine leaks
- deadlocks
- graceful shutdown

### Golden explanation

If asked:

> Why not just start a goroutine for every job?

Answer:

Because goroutines are lightweight, not free. Unlimited goroutine creation can create memory pressure, scheduling overhead and overload downstream resources. A worker pool provides bounded concurrency and backpressure.

---

## 4. Context vs defer

Remember:

```text
context → controls lifetime of work
defer   → guarantees cleanup when function exits
```

Example:

```go
func Handler(ctx context.Context) error {
    resource, err := acquire(ctx)
    if err != nil {
        return err
    }

    defer resource.Close()

    return process(ctx, resource)
}
```

---

## 5. DI question

If asked:

> How does Go handle dependency injection?

Answer:

Go does not require a heavyweight DI framework. Dependencies can be passed explicitly through constructors and interfaces.

```go
type Service struct {
    repo Repository
}

func NewService(repo Repository) *Service {
    return &Service{repo: repo}
}
```

This makes dependencies explicit and easy to replace in tests.

Important:

> Go is not preventing dependency injection. Its composition model makes manual/constructor injection simple enough that a framework is often unnecessary.

---

## 6. Benchmarking vs profiling

Interview answer:

> Benchmarking measures how an implementation performs under controlled repeated execution. Profiling identifies where CPU, memory, blocking or synchronization costs are actually occurring.

Commands:

```bash
go test -bench=.
go test -race ./...
go test -cpuprofile=cpu.out -bench=.
go vet ./...
```

---

## 7. Production API checklist

For a Go backend, discuss:

```text
Authentication
Authorization
Validation
Timeouts
Context propagation
Database pooling
Transactions
Caching
Rate limiting
Retries
Idempotency
Logging
Metrics
Tracing
Graceful shutdown
Testing
Race detection
```

---

## 8. System design checklist

When given a system-design problem:

### Step 1 — Requirements

Clarify:
- users
- traffic
- latency
- consistency
- availability
- data volume

### Step 2 — API

Define major operations.

### Step 3 — Storage

Choose:
- SQL
- NoSQL
- cache
- object storage
- search

### Step 4 — Scaling

Discuss:
- load balancing
- horizontal scaling
- partitioning
- caching

### Step 5 — Reliability

Discuss:
- timeouts
- retries
- circuit breaking where appropriate
- idempotency
- queues
- dead-letter handling

### Step 6 — Observability

Discuss:
- logs
- metrics
- traces
- alerts

### Step 7 — Failure modes

Ask what happens when:
- DB is down
- cache is down
- service is slow
- queue duplicates messages
- network partitions occur

---

## 9. Common Go traps

### Trap 1 — nil map write

```go
var m map[string]int
m["x"] = 1 // panic
```

### Trap 2 — nil interface

An interface containing a typed nil pointer can itself be non-nil.

### Trap 3 — slice aliasing

Two slices can share a backing array.

### Trap 4 — sending on closed channel

Panics.

### Trap 5 — closing twice

Panics.

### Trap 6 — goroutine leak

A goroutine can remain blocked forever.

### Trap 7 — concurrent map access

Unsafe concurrent read/write can race.

### Trap 8 — ignoring errors

Go expects explicit error handling.

### Trap 9 — assuming context cancels automatically

Context cancellation only works if downstream operations observe `ctx.Done()` or use context-aware APIs.

### Trap 10 — using panic for normal errors

Expected business failures should normally be returned as errors.

---

## 10. High-value interview explanations

### Why Go?

Strong answer:

> Go gives me static typing and compile-time checking while keeping the language relatively small and straightforward. Its concurrency model with goroutines and channels is particularly useful for backend services, and the standard tooling makes formatting, testing, profiling and builds consistent. It also produces efficient compiled binaries and has a strong production ecosystem.

### Why slices instead of arrays?

> Arrays have a fixed length that is part of their type. Slices provide a flexible view over an underlying array and are therefore the normal collection abstraction for variable-length sequences.

### Why interfaces?

> Interfaces let code depend on behavior rather than concrete implementations. Because Go uses implicit interface satisfaction, types don't need to declare that they implement an interface.

### Why context?

> Context propagates cancellation, deadlines and request-scoped metadata across API boundaries, especially from an incoming request to downstream services and databases.

### Why worker pool?

> To bound concurrency and prevent uncontrolled goroutine creation or downstream overload.

---

## 11. Interview-level coding habits

When solving a problem:

1. Clarify the problem.
2. State brute force briefly.
3. Identify the pattern.
4. Explain the optimized approach.
5. State time/space complexity.
6. Code.
7. Test with normal case.
8. Test edge cases.
9. Discuss trade-offs.

For Go specifically, write idiomatic code rather than trying to reproduce Java/C++ syntax.

---

## 12. Final mastery checklist

You are ready for a Go interview when you can explain, without memorized wording:

### Language
- [ ] static typing
- [ ] compilation
- [ ] packages
- [ ] exports
- [ ] structs
- [ ] pointers
- [ ] methods
- [ ] interfaces
- [ ] errors
- [ ] defer/panic/recover
- [ ] generics

### Runtime
- [ ] stack/heap
- [ ] escape analysis
- [ ] GC
- [ ] slices
- [ ] maps

### Concurrency
- [ ] goroutines
- [ ] channels
- [ ] select
- [ ] worker pools
- [ ] scheduler
- [ ] mutex
- [ ] atomic
- [ ] context
- [ ] cancellation
- [ ] deadlocks/leaks

### Tooling
- [ ] gofmt
- [ ] go test
- [ ] race detector
- [ ] benchmarks
- [ ] profiling
- [ ] go vet
- [ ] modules

### Backend
- [ ] HTTP
- [ ] middleware
- [ ] DB
- [ ] transactions
- [ ] caching
- [ ] retries
- [ ] rate limiting
- [ ] graceful shutdown

### Distributed systems
- [ ] queues
- [ ] idempotency
- [ ] eventual consistency
- [ ] CAP
- [ ] load balancing
- [ ] observability
- [ ] backpressure

### DSA
- [ ] slices/maps
- [ ] two pointers
- [ ] sliding window
- [ ] stack/queue
- [ ] heap
- [ ] BFS/DFS
- [ ] binary search
- [ ] sorting
- [ ] complexity
