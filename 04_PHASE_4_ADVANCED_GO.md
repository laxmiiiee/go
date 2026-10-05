# Phase 4 — Advanced Go

## 1. Generics

Generics allow functions/types to work across multiple types while retaining compile-time type safety.

```go
func Max[T int | float64](a, b T) T {
    if a > b {
        return a
    }
    return b
}
```

Use generics when the algorithm is genuinely type-independent.

Do not introduce generics merely to look advanced.

---

## 2. Generic constraints

A constraint specifies which types are allowed.

Conceptually:

```text
T must support the operations the function needs
```

The `any` constraint means any type.

---

## 3. Testing

Go has built-in testing support.

```go
func TestAdd(t *testing.T) {
    got := Add(2, 3)
    want := 5

    if got != want {
        t.Fatalf("got %v want %v", got, want)
    }
}
```

Run:

```bash
go test ./...
```

---

## 4. Table-driven tests

Go commonly uses table-driven tests:

```go
tests := []struct {
    name string
    a    int
    b    int
    want int
}{
    {"positive", 2, 3, 5},
    {"zero", 0, 3, 3},
}
```

This makes edge cases easy to expand.

---

## 5. Benchmarks

Benchmarks live in `_test.go` files.

```go
func BenchmarkWork(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Work()
    }
}
```

Run:

```bash
go test -bench=.
```

Benchmarking answers:

> How does this code perform under repeated measurement?

---

## 6. Profiling

Profiling answers:

> Where is the program actually spending resources?

Common profiles:
- CPU
- heap/memory
- goroutines
- blocking
- mutex contention

A common command is:

```bash
go test -cpuprofile=cpu.out -bench=.
```

Then inspect using Go's profiling tooling.

---

## 7. Benchmarking vs profiling

### Benchmarking

Compares measurable performance.

```text
implementation A: 100 ns/op
implementation B: 150 ns/op
```

### Profiling

Finds where resources are being consumed.

```text
CPU hot spot
memory allocation hot spot
lock contention
blocking
```

Mental model:

```text
Benchmark → "How fast?"
Profile    → "Where is the cost?"
```

---

## 8. Go tooling

Important tools:

```bash
go fmt
go test
go test -race
go vet
go build
go run
go mod tidy
go doc
```

A Go engineer should be comfortable using tooling instead of manually reasoning about performance from source code alone.

---

## 9. Error wrapping

Modern Go supports wrapping errors:

```go
return fmt.Errorf("get user: %w", err)
```

Then:

```go
errors.Is(err, target)
errors.As(err, &target)
```

Wrapping preserves the underlying error while adding context.

---

## 10. Error design

Good errors explain:
- what operation failed
- useful context
- underlying cause when relevant

Avoid swallowing errors:

```go
if err != nil {
    // ignoring it
}
```

unless the omission is deliberate.

---

## 11. defer and resource safety

Use defer near resource acquisition:

```go
file, err := os.Open(path)
if err != nil {
    return err
}
defer file.Close()
```

This keeps cleanup close to the acquisition site.

---

## 12. panic/recover in services

Do not use panic for expected business errors.

`recover` can be useful at a controlled boundary, such as protecting a server process from one unexpected panic taking down unrelated work.

The exact policy depends on the service architecture.
