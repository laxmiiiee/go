# Phase 1 — Go Fundamentals

## 1. Static typing

Go is statically typed.

The type of a variable is known and checked by the compiler before the program runs.

```go
var age int = 25
var name string = "Lakshmi"
```

This is different from JavaScript:

```js
let age = 25;
age = "hello";
```

JavaScript allows the variable to change runtime type. Go does not allow arbitrary type changes.

### Why it matters

Static typing:
- catches many mistakes before execution
- makes APIs easier to reason about
- improves tooling and refactoring
- makes large codebases safer

Go still has type inference:

```go
age := 25
```

The compiler infers `int`; this does NOT make Go dynamically typed.

---

## 2. Compiled language

Go source code is compiled into machine code.

Typical flow:

```text
.go source
   ↓
Go compiler
   ↓
machine code / executable
   ↓
operating system + CPU
```

This differs from languages where source is primarily interpreted at runtime.

Compilation catches type and many syntax errors before the executable is produced.

Useful commands:

```bash
go run main.go
go build
go build -o app
```

`go run` is convenient during development. `go build` produces a compiled executable.

---

## 3. Package

A package is Go's unit of organization and compilation.

```go
package main
```

`package main` is special: together with a `main()` function, it defines an executable program.

Other packages can provide reusable functionality.

---

## 4. main package and main function

```go
package main

func main() {
}
```

Two different concepts:

- `package main` tells Go this package builds as an executable.
- `func main()` is the entry point of that executable.

Execution starts from `main()`.

---

## 5. fmt and fmt.Println

```go
import "fmt"

func main() {
    fmt.Println("Hello")
}
```

`fmt` is part of Go's standard library and provides formatting and I/O helpers.

`Println` is an exported function in the `fmt` package.

The dot means:

```text
package.Function
```

---

## 6. Exported vs unexported identifiers

Go uses capitalization for visibility.

```go
func Calculate() {}
func calculate() {}
```

- `Calculate` is exported.
- `calculate` is unexported.

An exported identifier can be accessed from another package.

This applies to functions, types, variables, constants, methods and fields.

```go
type User struct {
    Name string // exported
    age  int    // unexported
}
```

This is why standard library APIs often have capitalized names.

---

## 7. Variables and constants

```go
var x int = 10
var name = "Go"

x := 10
```

Short declaration `:=` is commonly used inside functions.

Constants:

```go
const Pi = 3.14
```

A constant is a value known at compile time where applicable.

---

## 8. Basic types

Common types:

```text
bool
string
int
int8/int16/int32/int64
uint...
float32/float64
complex64/complex128
byte
rune
```

Important aliases:

```go
byte // alias for uint8
rune // alias for int32
```

A `rune` represents a Unicode code point.

---

## 9. Arrays vs slices

Array:

```go
var a [3]int
```

The length is part of the array's type.

```text
[3]int != [4]int
```

Slice:

```go
s := []int{1, 2, 3}
```

A slice is a flexible view over an underlying array.

Important slice concepts:

```text
pointer
length
capacity
```

```go
len(s)
cap(s)
```

Appending:

```go
s = append(s, 4)
```

If capacity is insufficient, Go allocates a new backing array.

---

## 10. Maps

```go
ages := map[string]int{
    "A": 25,
}
```

Lookup:

```go
age, ok := ages["A"]
```

The `ok` value distinguishes:
- key exists
- key does not exist

Delete:

```go
delete(ages, "A")
```

A map is a reference-like runtime data structure; assignment/copying does not deep-copy its entries.

---

## 11. Structs

Go does not primarily use classes.

Instead, data is commonly represented using structs:

```go
type User struct {
    Name string
    Age  int
}
```

Create:

```go
u := User{
    Name: "A",
    Age: 25,
}
```

Structs group related fields.

---

## 12. Pointers

```go
x := 10
p := &x

fmt.Println(*p)
```

- `&x` gets the address.
- `*p` dereferences the pointer.

A pointer lets a function operate on the original value rather than a copy.

```go
func change(x *int) {
    *x = 100
}
```

Go has pointers but does not expose pointer arithmetic like C.

---

## 13. Methods

A method is a function associated with a receiver.

```go
type User struct {
    Name string
}

func (u User) Greet() string {
    return "Hello " + u.Name
}
```

Pointer receiver:

```go
func (u *User) Rename(name string) {
    u.Name = name
}
```

Use pointer receivers when mutation is required or copying the value is undesirable.

---

## 14. Interfaces

Go interfaces describe behavior.

```go
type Speaker interface {
    Speak()
}
```

A type satisfies an interface implicitly by implementing its methods.

There is no `implements` keyword.

This enables loose coupling.

---

## 15. The nil interface trap

An interface internally has:
- a dynamic type
- a dynamic value

An interface can be non-nil while containing a nil pointer.

Conceptually:

```text
interface
 ├── type = *User
 └── value = nil
```

Therefore:

```go
var p *User = nil
var x interface{} = p

x != nil
```

This is a classic interview trap.

---

## 16. Errors

Go commonly returns errors explicitly:

```go
result, err := doSomething()

if err != nil {
    return err
}
```

This makes failure part of the function's normal control flow.

---

## 17. defer

`defer` schedules a function call for execution when the surrounding function returns.

```go
func read() {
    file := open()
    defer file.Close()
}
```

Multiple defers execute in LIFO order:

```text
defer A
defer B
defer C

return

C
B
A
```

`defer` is excellent for cleanup.

---

## 18. panic and recover

`panic` represents an abnormal failure that starts stack unwinding.

`recover` can intercept a panic, but only when called from a deferred function during panic handling.

Do not use panic as normal error handling.

---

## 19. Modules

A Go project normally has a `go.mod`.

```bash
go mod init example.com/project
go mod tidy
```

Modules manage dependencies and version information.

---

## 20. Formatting and tooling

Go strongly standardizes formatting:

```bash
gofmt
```

Useful commands:

```bash
go fmt
go test
go vet
go build
go run
go mod tidy
go doc
```

A major Go philosophy is that tooling should be consistent across projects.
