# Phase 2 — Go Internals, Memory, Interfaces and GC

## 1. Stack vs heap

Go uses both stack and heap memory.

A function's local data may initially live on the goroutine stack.

If the compiler determines a value must survive beyond the current stack frame, it may escape to the heap.

Do not think:

> pointers always mean heap.

That is false.

---

## 2. Escape analysis

Escape analysis determines whether values can remain on the stack or need heap allocation.

Inspect it with:

```bash
go build -gcflags="-m"
```

The compiler can often optimize allocations automatically.

Typical causes of escaping include:
- returning a pointer to a local value
- storing a value somewhere whose lifetime outlives the current function
- certain interface conversions
- closures capturing variables

The exact result is compiler-dependent.

---

## 3. Garbage collection

Go has a garbage collector.

The GC identifies memory that is no longer reachable and reclaims it.

The developer generally does not manually free memory.

The goal is to provide automatic memory management while keeping latency acceptable.

---

## 4. Value semantics

Structs are values.

```go
u2 := u1
```

This copies the struct's fields.

But reference-like structures such as slices and maps contain references to underlying runtime data, so copying the slice/map value does not deep-copy the underlying contents.

This distinction is extremely important in interviews.

---

## 5. Slice internals

Conceptually a slice contains:

```text
pointer → backing array
length
capacity
```

Two slices can refer to the same backing array.

Therefore:

```go
a := []int{1, 2, 3}
b := a
b[0] = 99
```

`a[0]` can also become `99`.

Appending can change this behavior if a new backing array is allocated.

---

## 6. Map internals

Maps are implemented by the runtime using hash-table structures.

Important interview facts:
- map lookup is designed to be efficient
- map iteration order should not be relied upon
- concurrent read/write access requires synchronization
- a nil map can be read from, but writing to a nil map panics

```go
var m map[string]int

fmt.Println(m["x"]) // zero value

m["x"] = 1 // panic
```

Initialize with:

```go
m = make(map[string]int)
```

---

## 7. Zero values

Go deliberately gives variables useful zero values.

Examples:

```text
int     → 0
float   → 0
bool    → false
string  → ""
pointer → nil
slice   → nil
map     → nil
interface → nil
```

Good Go APIs often take advantage of zero values.

---

## 8. Interfaces internally

An interface value is conceptually associated with:
- dynamic type
- dynamic value

This explains the nil-interface trap and why interface comparisons have rules.

Interfaces enable abstraction without inheritance.

---

## 9. Garbage collection and performance

GC is automatic, but allocation still matters.

Excessive short-lived heap allocations can increase:
- allocation cost
- GC work
- latency
- memory pressure

Optimization should be evidence-driven.

Use benchmarks and profiles rather than guessing.

---

## 10. Race conditions

A race occurs when multiple goroutines access shared state concurrently and at least one access is a write, without proper synchronization.

Use:

```bash
go test -race ./...
```

The race detector is a key production-quality tool.

---

## 11. Mutex

```go
var mu sync.Mutex
var count int

mu.Lock()
count++
mu.Unlock()
```

Prefer:

```go
mu.Lock()
defer mu.Unlock()
```

when the critical section is straightforward.

A mutex protects shared state.

---

## 12. RWMutex

`sync.RWMutex` supports:
- multiple concurrent readers
- exclusive writer access

Use it only when the workload benefits from that complexity.

A mutex is often simpler and sufficient.
