# Phase 5 — Production Backend Go

## 1. HTTP service architecture

A typical Go backend:

```text
Client
  ↓
HTTP Router
  ↓
Handler
  ↓
Service / Business Logic
  ↓
Repository / DB
  ↓
PostgreSQL
```

Keep responsibilities separated.

### Handler

Deals with:
- HTTP request
- authentication context
- validation
- response status/body

### Service

Deals with:
- business rules
- orchestration
- transactions where appropriate

### Repository

Deals with:
- persistence
- SQL/data access

---

## 2. Request lifecycle

```text
request
 ↓
middleware
 ↓
handler
 ↓
service
 ↓
repository
 ↓
database
 ↓
response
```

Context should propagate through this chain.

---

## 3. Authentication and authorization

Authentication:
> Who are you?

Authorization:
> Are you allowed to perform this action?

A Go service can implement:
- JWT validation
- OAuth/OIDC integration
- RBAC
- middleware-based access control

Do not mix authentication with authorization logic unnecessarily.

---

## 4. Middleware

Middleware wraps request handling.

Conceptually:

```text
logging
  ↓
auth
  ↓
rate limiting
  ↓
handler
```

Middleware is useful for cross-cutting concerns.

---

## 5. PostgreSQL

Backend Go services commonly use PostgreSQL through a driver or database library.

Important production topics:
- connection pooling
- transactions
- indexes
- query performance
- isolation
- timeouts
- cancellation using context

---

## 6. Context and database calls

Pass context into database operations where supported:

```go
db.QueryContext(ctx, query)
```

If the HTTP request is cancelled, downstream DB work can often be cancelled too.

This prevents wasted work.

---

## 7. Transactions

A transaction groups operations into an atomic unit.

```text
BEGIN
 ↓
operation 1
 ↓
operation 2
 ↓
COMMIT
```

If something fails:

```text
ROLLBACK
```

Use transactions when multiple writes must maintain a consistent business invariant.

---

## 8. Caching

A cache reduces expensive repeated work.

Typical architecture:

```text
request
 ↓
cache
 ├── hit → response
 └── miss → DB → cache → response
```

Interview concerns:
- TTL
- invalidation
- stale data
- cache stampede
- memory limits
- distributed cache consistency

---

## 9. Timeouts

Production calls should not wait forever.

Use context deadlines/timeouts for:
- HTTP calls
- DB operations
- internal service calls

A timeout is part of resource management.

---

## 10. Rate limiting

Rate limiting protects services from excessive traffic.

Possible strategies:
- token bucket
- leaky bucket
- fixed/sliding windows

In a distributed system, local in-memory rate limiting may not be enough if requests can reach multiple instances.

---

## 11. Logging

Production logs should contain useful structured information.

Useful fields:
- request ID
- operation
- latency
- status
- error
- relevant entity ID

Avoid logging secrets and sensitive information.

---

## 12. Graceful HTTP shutdown

A server should not abruptly kill in-flight requests.

Conceptual sequence:

```text
SIGTERM
 ↓
stop accepting new requests
 ↓
allow active requests to finish
 ↓
close DB/other resources
 ↓
exit
```

Use context deadlines so shutdown cannot hang forever.

---

## 13. Worker pools in backend services

Worker pools are useful for:
- background jobs
- bounded external API calls
- CPU/IO processing pipelines
- controlled batch work

They prevent unlimited concurrency.

Always think about:
- queue capacity
- worker count
- cancellation
- retries
- failed jobs
- backpressure
- shutdown

---

## 14. Dependency injection in Go

Go does not need a DI framework for ordinary dependency injection.

Prefer explicit dependencies:

```go
type Service struct {
    repo Repository
}

func NewService(repo Repository) *Service {
    return &Service{repo: repo}
}
```

The caller wires dependencies.

This is constructor injection.

### Why this works well

Go favors:
- interfaces
- small structs
- explicit constructors
- composition

This makes dependencies visible and testing easier.

DI is not "prevented" by Go. Rather, Go's simple language features make heavyweight DI frameworks less necessary.

---

## 15. Testing backend code

Inject dependencies so they can be replaced in tests.

Example:

```text
Handler
  ↓
Service interface
  ↓
Mock/fake repository
```

The goal is not to mock everything. Test business behavior and important boundaries.

---

## 16. API reliability

A production API should consider:
- validation
- authentication
- authorization
- timeouts
- retries
- idempotency
- rate limiting
- logging
- metrics
- tracing
- graceful shutdown

---

## 17. Retries

Retry only operations that are safe to retry or explicitly designed to be idempotent.

Use:
- bounded retries
- exponential backoff
- jitter

Otherwise many clients can retry simultaneously and amplify an outage.
