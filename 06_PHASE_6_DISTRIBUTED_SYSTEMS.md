# Phase 6 — Distributed Systems and System Design

## 1. Service-to-service communication

A Go service may communicate using:
- HTTP/REST
- gRPC
- messaging systems

Choose based on requirements rather than language preference.

---

## 2. Synchronous vs asynchronous

Synchronous:

```text
A → request → B
A waits
```

Asynchronous:

```text
A → queue → B
A can continue
```

Async systems improve decoupling but introduce:
- eventual consistency
- retries
- duplicate messages
- ordering concerns
- observability complexity

---

## 3. Idempotency

An operation is idempotent if repeating it produces the same intended final effect.

Example:

```text
PUT /user/123
```

can be designed to set a resource to a known state.

For payments or job processing, idempotency keys are commonly used to prevent duplicate effects.

---

## 4. Message processing

A robust consumer should consider:

```text
receive
 ↓
validate
 ↓
process
 ↓
acknowledge
```

If processing fails:
- retry
- dead-letter
- or explicitly record failure

Do not acknowledge a message before the required work is safely completed.

---

## 5. At-least-once delivery

At-least-once delivery can result in duplicate processing.

Therefore consumers should often be designed to be idempotent.

This is a common distributed-systems interview point.

---

## 6. CAP intuition

CAP concerns distributed systems under network partition.

You cannot simultaneously guarantee:
- strong consistency
- availability
- partition tolerance

when a partition occurs.

In practice, partition tolerance is required for distributed systems, so the design involves consistency/availability trade-offs.

---

## 7. Consistency

Strong consistency:
> reads observe the latest committed state according to the system's consistency model.

Eventual consistency:
> replicas may temporarily differ but converge.

Neither is universally better.

Choose based on business requirements.

---

## 8. Caching in distributed systems

Distributed caches introduce:
- cache invalidation
- stale data
- replication
- failure handling
- eviction
- stampede protection

Classic interview principle:

> Cache is an optimization, not the source of truth, unless deliberately designed otherwise.

---

## 9. Load balancing

Typical architecture:

```text
Clients
  ↓
Load Balancer
  ↓
Service instances
  ↓
Database/cache
```

Stateless services scale horizontally more easily.

If session state exists, consider externalizing it or using an appropriate session strategy.

---

## 10. Observability

Three major pillars:

```text
Logs
Metrics
Traces
```

Metrics answer:
> Is the system healthy?

Logs answer:
> What happened?

Traces answer:
> Where did this request spend time?

---

## 11. Backpressure

Backpressure prevents a fast producer from overwhelming a slower consumer.

Worker pools, bounded queues and rate limits are mechanisms for controlling pressure.

---

## 12. Failure design

For every dependency ask:

- What if it is slow?
- What if it times out?
- What if it returns errors?
- What if it returns malformed data?
- What if it is unavailable?
- What if retries duplicate work?

This mindset is critical for senior-level interviews.
