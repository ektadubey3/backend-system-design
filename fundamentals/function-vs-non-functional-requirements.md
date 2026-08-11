# Functional vs Non-Functional Requirements

Requirements are architecture inputs. A system-design interview should not begin with a database, cache, or queue; it should begin by identifying the **user-visible behavior** and the **quality constraints that can change the design**.

## Interview TL;DR

1. **Functional requirements (FRs)** describe the behaviors the system must support.
2. **Non-functional requirements (NFRs)** describe measurable qualities or constraints: latency, availability, durability, consistency, security, scale, retention, cost, and geography.
3. Identify the **critical user journey** and **hard invariants** first.
4. Replace vague words such as “fast,” “highly available,” and “scalable” with measurable assumptions.
5. Ask only clarifying questions whose answers could change the architecture.
6. Explicit non-goals prevent the design from expanding without limit.

## Mental Model

```text
What must happen?         → Functional requirements
How well must it happen?  → Non-functional requirements
What must never happen?   → Invariants
What can we ignore now?   → Non-goals
```

## Functional Requirements

Functional requirements describe externally visible behavior.

For a URL shortener:

- create a short link;
- redirect a short code;
- optionally support expiry;
- optionally support custom aliases;
- record click analytics.

For a chat system:

- send a message;
- receive messages;
- fetch history;
- show delivery/read state;
- support multiple devices.

Prioritize the flows that actually determine the architecture.

## Non-Functional Requirements

### Scale

State the dimensions that matter:

```text
read QPS
write QPS
concurrent connections
dataset size
growth/day
largest tenant
peak multiplier
```

“10 million users” is usually less useful than “80,000 peak reads/sec and 3,000 writes/sec.”

### Latency

Prefer percentiles:

```text
redirect p99 < 100 ms
message-send p95 < 200 ms
```

Treat these as explicit product assumptions, not universal thresholds.

### Availability

Define the operation:

```text
redirect path: 99.99%
analytics dashboard: 99.9%
```

### Consistency and Freshness

Name the guarantee:

- linearizable decision;
- read-your-writes;
- monotonic reads;
- bounded staleness;
- eventual convergence.

Example:

```text
inventory reservation → current authoritative state
search index          → may lag by 5 seconds
analytics             → may lag by minutes
```

### Durability

Ask:

> After the client receives success, how much acknowledged data can we lose?

Express this with an RPO or an equivalent product statement.

### Security and Abuse

Include architecture-changing constraints:

- authentication/authorization;
- tenant isolation;
- sensitive data;
- auditability;
- rate limiting;
- abuse detection;
- data residency.

### Geography

Ask:

- one region or global?
- where are users?
- where may data be stored?
- must writes survive region loss?

### Retention

Retention directly changes storage design.

## Invariants

Examples:

```text
one seat cannot be sold twice
one idempotency key cannot create two payments
a ledger entry is never silently overwritten
```

Invariants are more actionable than broad labels such as “strong consistency.”

## Non-Goals

> “For the first version, I will design link creation and redirect. Rich analytics ranking and custom domains are out of scope unless we have time.”

## Example — Notification Service

### Functional

- accept a notification request;
- deliver email, SMS, or push;
- query delivery status.

### Non-functional

- request acceptance p99 < 200 ms;
- delivery is asynchronous;
- no silent loss after durable acceptance;
- downstream effects are idempotent;
- promotional notifications may be delayed during overload;
- security notifications receive higher priority.

These requirements immediately imply durable queuing, idempotency, priority, retry policy, and backlog observability.

## Common Mistakes

- asking questions whose answers do not affect design;
- treating availability as one global number;
- naming technology as a requirement;
- forgetting failure behavior;
- estimating quantities that do not change a decision.

## 2-Minute Interview Answer

> “I start with the critical user journeys, then make the quality constraints measurable. I identify strict invariants, read/write and connection scale, latency and availability targets, retention, geography, security, and acceptable staleness. I also state non-goals. Those requirements become decision drivers rather than jumping directly to a database or queue.”

## Senior-Level Follow-ups

- how an NFR becomes an SLI/SLO;
- what happens when NFRs conflict;
- which operation wins during overload;
- how requirements differ by operation;
- how 10× scale changes the design.
