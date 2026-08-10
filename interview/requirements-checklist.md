# Requirements Checklist

## Functional

Ask for the small set of user behaviors that define the system.

Examples:

- create/read/update/delete;
- search;
- feed generation;
- send/receive events;
- upload/download;
- reserve/pay;
- realtime presence;
- reporting.

## Non-functional

### Scale

- DAU/MAU
- peak RPS
- read/write ratio
- object size
- storage growth
- concurrent connections

### Performance

- p50/p95/p99 latency
- throughput
- asynchronous completion tolerance

### Reliability

- availability target
- durability
- RPO/RTO
- graceful degradation

### Consistency

Ask per operation:

- Must reads observe the latest write?
- Can stale reads be tolerated?
- Is ordering required globally or per entity?
- Can duplicate processing occur?

### Security

- authentication
- authorization
- tenant isolation
- encryption
- abuse/rate limits
- auditability

### Geography

- user distribution
- data residency
- active-active vs active-passive expectations

## Non-goals

State them explicitly. Good design interviews are scoped exercises, not attempts to design every supporting subsystem.
