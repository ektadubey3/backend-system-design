# Event-Driven Architecture, CQRS, and Event Sourcing

## TL;DR

Event-driven architecture communicates committed facts asynchronously. CQRS separates command and query models when their shapes or scaling differ. Event sourcing makes an event stream the authoritative state history. These are independent choices: an outbox-fed event-driven system need not be event-sourced, and CQRS can use an ordinary transactional database.

## Event-driven architecture

A producer publishes an immutable fact after its local transaction; independent consumers update their own state. Benefits include temporal decoupling, fan-out, and replayable integration. Costs include lag, duplicates, ordering scope, schema lifecycle, DLQs, and harder end-to-end reasoning.

Use commands for requested ownership (`ReserveSeat`) and events for completed facts (`SeatHeld`). Avoid events named as commands or vague `EntityUpdated` envelopes that force consumers to query mutable state.

## CQRS

Command Query Responsibility Segregation separates the write model that enforces invariants from read models optimized for queries. At a small scale this can be separate classes/tables inside one application. At larger scale, outbox events build caches, indexes, or denormalized stores.

Read models are derived. Define freshness objectives, version/gap detection, rebuild, authorization, and what the UI shows before propagation. Do not let a stale projection authorize a sensitive write.

## Event sourcing

An event-sourced aggregate appends validated domain events and reconstructs state by folding them. Expected version enforces optimistic concurrency:

```text
load events through version 17
decide command -> new events
append only if stream version is still 17
```

Snapshots accelerate reconstruction but are caches tied to an event position. Event schemas and business interpretation must remain replayable. Corrections append new facts; they do not rewrite history casually.

## When event sourcing helps

- history and temporal queries are core product value;
- decisions need audit/reconstruction beyond ordinary change logs;
- domain transitions are naturally modeled as immutable facts;
- rebuilding several projections is expected;
- the team can own schema evolution, replay, and operational tooling.

It is a poor default for simple CRUD, large mutable blobs, or teams without replay/debug tooling. Database audit tables plus outbox may meet the requirement more cheaply.

## Consistency and process managers

One aggregate stream can enforce its own invariant. A workflow across streams/services needs a saga/process manager that persists progress, uses idempotent commands, reacts to events, times out, and compensates. Event sourcing does not create an atomic transaction across aggregates.

## Failure modes

- Publishing before local commit creates events for rolled-back state.
- Event name encodes implementation rather than stable business meaning.
- Rebuild re-sends emails/payments because projections and effects are mixed.
- Event upcaster depends on current external state and makes replay nondeterministic.
- CQRS projection lag exposes stale authorization or inventory.
- Event store retention/deletion obligations were never designed.

## Interview prompts

- Is the event log authoritative or an integration copy?
- Which invariant fits inside one command/aggregate?
- Can every projection rebuild without repeating external effects?
- Why does this need event sourcing rather than outbox plus audit history?

## Two-minute answer

Use event-driven integration for committed facts through an outbox, accepting at-least-once delivery and explicit contracts. Add CQRS when write invariants and read shapes genuinely differ; derived views have a freshness and rebuild plan. Choose event sourcing only when immutable history and temporal reconstruction justify permanent event/schema/replay complexity. Append under expected stream version, snapshot by position, keep irreversible effects outside projection replay, and coordinate cross-aggregate work with a durable saga rather than assuming the log makes it atomic.

## Further study

- [Messaging systems](../messaging/README.md)
- [SQL transactions: outbox and saga](../databases/sql/transactions.md)

