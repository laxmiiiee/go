# Phase 7 — DSA in Go

## 1. Core Go data structures for DSA

You should be comfortable with:

```text
[]T              slice
map[K]V          hash map
struct           custom data
string            immutable byte sequence abstraction
[]byte            mutable byte data
container/heap   heap utilities
```

For interviews, slices are the primary array representation.

---

## 2. Two pointers

Typical cases:

### Opposite ends

```text
left →       ← right
```

Useful for:
- sorted arrays
- palindrome checks
- container problems

### Same direction

```text
read →
write →
```

Useful for:
- removing duplicates
- moving elements
- in-place filtering

---

## 3. Sliding window

Maintain a range:

```text
left ........ right
```

Expand `right`, and move `left` when the window violates a condition.

Common problems:
- longest substring
- maximum/minimum window
- subarray constraints

---

## 4. Hash map pattern

Use a map when you need:
- fast membership
- frequency counting
- lookup by key
- complement lookup

Typical complexity:

```text
average O(1) lookup
```

---

## 5. Stack

Use a slice:

```go
stack := []int{}

stack = append(stack, x)

top := stack[len(stack)-1]

stack = stack[:len(stack)-1]
```

Common problems:
- parentheses
- monotonic stack
- DFS
- expression processing

---

## 6. Queue

A slice can represent a queue, but avoid repeatedly removing from the front with expensive shifting in performance-sensitive code.

A common pattern uses an index:

```go
queue := []int{...}
head := 0

for head < len(queue) {
    x := queue[head]
    head++
}
```

---

## 7. Heap

Go provides:

```go
container/heap
```

A heap is useful for:
- top K
- scheduling
- priority queues
- merging sorted streams

---

## 8. BFS

Typical queue-based approach:

```text
queue
 ↓
take node
 ↓
visit neighbors
 ↓
push unvisited neighbors
```

Complexity for graph traversal:

```text
O(V + E)
```

---

## 9. DFS

Can be recursive or iterative.

Recursive DFS is simple but deep graphs can make recursion depth a concern.

---

## 10. Binary search

Use when the search space is ordered or when a monotonic condition exists.

Classic structure:

```text
left
right

while left <= right
```

Be careful with:
- boundaries
- midpoint
- inclusive/exclusive ranges
- integer overflow in languages where it matters

---

## 11. Sorting

Use the standard library when allowed rather than implementing sorting manually.

```go
sort.Ints(nums)
```

For custom ordering, use appropriate `sort` APIs.

---

## 12. Strings

Go strings are byte sequences representing UTF-8 encoded text by convention.

This matters:

```go
len("é")
```

counts bytes, not necessarily human-visible characters.

For Unicode code points, use `rune`.

This distinction is a common Go interview question.

---

## 13. Complexity

Always state:

```text
Time: O(...)
Space: O(...)
```

Then explain why.

Do not assume a library call is O(1) without knowing its behavior.
