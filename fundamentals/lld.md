# Low-Level Design (LLD)

Low-Level Design describes the internal design of a component: interfaces, data structures, state transitions, concurrency rules, error contracts, and implementation boundaries.

For this backend system-design repository, LLD is supplementary. Do not let class diagrams replace distributed-system reasoning in an HLD interview.

## Interview TL;DR

1. LLD is broader than memorizing OOP patterns.
2. Start from responsibilities and invariants, then define interfaces.
3. Model workflow state transitions explicitly.
4. Prefer composition and cohesive modules.
5. Concurrency and failure behavior belong in LLD when a component owns mutable state.
6. Use a design pattern only when it solves a demonstrated problem.
7. Design error, timeout, and retry contracts with the success path.
8. Keep persistence boundaries explicit.
9. Make the design testable.
10. Avoid speculative abstractions.

## What LLD Can Include

- interfaces and method contracts;
- classes/modules/packages;
- state machines;
- data structures;
- algorithms;
- validation;
- local concurrency;
- schema details;
- error model;
- dependency boundaries;
- sequence diagrams;
- tests.

## Example — Order State Machine

```text
CREATED
   ↓
PAYMENT_PENDING
   ├─→ PAID
   └─→ PAYMENT_FAILED
          ↓
       CANCELLED
```

Invalid transitions must be rejected. Here `PAID` means capture is confirmed; authorization alone does not establish it. A timeout is an unknown outcome, not `PAYMENT_FAILED`: remain pending and query/reconcile. This minimal model omits refund and cancellation-after-capture flows, which require explicit states and compensation policies.

## Interfaces

```java
interface OrderRepository {
    Optional<Order> findById(OrderId id);
    boolean saveIfVersionMatches(Order order, long expectedVersion);
}

interface PaymentClient {
    PaymentResult capture(
        OrderId orderId,
        Money amount,
        IdempotencyKey key,
        Duration timeout
    );
}
```

A useful contract makes remote-failure semantics visible.

## Domain Invariant

```java
final class Order {
    private OrderStatus status;

    void markPaid() {
        if (status != OrderStatus.PAYMENT_PENDING) {
            throw new InvalidOrderTransition();
        }
        status = OrderStatus.PAID;
    }
}
```

## Concurrency

If two requests can modify the same state, define:

- atomic DB update;
- optimistic version;
- lock;
- serialized executor;
- immutable event stream.

Do not assume single-threaded execution if production does not guarantee it. The in-memory `markPaid()` check protects one object, not two processes reading the same row. The repository contract `saveIfVersionMatches(order, expectedVersion)` returns false on a version conflict; a blind `save(order)` would not enforce this boundary.

For this single-row invariant, a PostgreSQL conditional update can enforce the transition:

```sql
UPDATE orders
SET status = 'PAID', version = version + 1
WHERE id = :order_id
  AND status = 'PAYMENT_PENDING'
  AND version = :expected_version
RETURNING id, status, version;
```

Call this only after verified capture. One returned row means the transition was applied; no row means reload and distinguish missing order, duplicate completion, or conflicting state. Record the provider capture ID under an appropriate uniqueness constraint. If publishing an event, insert the outbox record in the same transaction. At PostgreSQL Read Committed, a conflicting updater waits and rechecks the predicate against the updated row. See [PostgreSQL transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html).

Do not hold a database transaction open across a provider call. Persist intent, perform the call with stable idempotency, then commit the result; reconciliation handles a crash between those stages. This example does not enforce multi-row invariants or make external effects atomic. See [database consistency boundaries](../databases/consistency-boundaries.md).

## Error Model

Differentiate:

```text
validation error
conflict
not found
dependency timeout
retryable infrastructure error
permanent business failure
```

Do not convert every error to HTTP 500.

## Persistence Boundary

Keep ownership explicit:

```text
domain/service
     ↓
repository/data access
     ↓
database
```

Do not add layers mechanically.

## Design Patterns

Useful when they solve a concrete variation:

- Strategy;
- Adapter;
- Factory;
- Observer/eventing.

Avoid defaulting to Singleton or deep inheritance hierarchies.

## Testing

Separate:

- deterministic rules;
- external I/O;
- time/randomness;
- persistence;
- side effects.

Test invalid transitions and concurrency conflicts.

## HLD vs LLD

| HLD | LLD |
|---|---|
| Service/data boundaries | Module/class boundaries |
| Network/storage choices | Interfaces/algorithms |
| Scaling/failure domains | Local concurrency/errors |
| Distributed consistency | State invariants |
| Evolution | Implementation extensibility |

## Common Mistakes

- “LLD = UML”;
- forcing every design into inheritance;
- patterns before responsibilities;
- giant service classes;
- mutable state with no concurrency rule;
- hidden remote calls that look cheap/local.

## 2-Minute Interview Answer

> “For LLD I start with responsibilities and invariants, define the state model and interfaces, then separate domain logic from external I/O. I make concurrency, timeout, error, and persistence semantics explicit. I prefer composition and simple modules, adding patterns only for a concrete variation or integration problem.”
