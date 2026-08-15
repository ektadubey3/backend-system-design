# Schema Evolution and Replay

## TL;DR

An event is an API to consumers you may not control, including consumers that are offline during a deployment. Favor additive evolution, explicit compatibility checks, stable semantics, and tolerant readers. Replay is a product and operations feature: define source position, side-effect suppression, rate limits, and the state that will be rebuilt.

## Compatibility has a direction

- **Backward compatible:** a new consumer can read old records.
- **Forward compatible:** an old consumer can read records produced by a new producer.
- **Full compatibility:** both directions hold across the supported version window.

During rolling deployment, old and new code coexist. During replay, today's consumer may read years-old records. Test the directions the rollout and retention model actually require.

## Safer evolution

Prefer:

- adding optional fields with defined defaults;
- adding new event types rather than changing the meaning of an old type;
- stable field identifiers in schema formats that use them;
- an envelope with event ID, type, schema version, producer, occurrence time, and business key;
- consumer-driven contract tests and registry compatibility gates.

Treat renaming as add-copy-migrate-remove, not an atomic edit. Removing a field requires evidence that every supported consumer stopped depending on it and that retained records remain readable.

Avoid “optional but semantically required.” If a missing field changes behavior, specify the default in business terms. Do not reuse an old field for a new meaning.

## Event time and processing time

Occurrence time describes when the business fact happened; ingestion time describes when the platform accepted it; processing time describes when a consumer handled it. Late and out-of-order records make these differ. Analytics windows, expirations, and reconciliation must choose deliberately.

Clock timestamps are not a substitute for per-entity versions when causal order matters. See [Time and logical clocks](../distributed-systems/time-and-logical-clocks.md).

## Replay modes

| Mode | Purpose | Guardrail |
|---|---|---|
| Rebuild projection | regenerate derived database/index | write to a new version, validate, then switch |
| Repair range | correct affected keys or positions | bounded selection and audit trail |
| Bootstrap consumer | create state for a new subscriber | throttle so live traffic remains healthy |
| Re-drive failures | retry corrected poison messages | preserve idempotency keys and attempt history |

Replaying a fact must not blindly repeat irreversible side effects. Separate pure projection handlers from effectful notification/payment handlers, or pass an explicit replay mode that disables effects. Do not silently make behavior depend on the current wall clock if historical reconstruction should be deterministic.

## Retention and snapshots

Long retention improves auditability and reconstruction but costs storage and increases privacy exposure. A compacted stream retains the latest value per key, which is useful for current state but not a complete history. Snapshots reduce rebuild time, yet need a precise stream position so replay resumes without gaps.

Estimate recovery:

```text
rebuild time ~= records to replay / sustainable replay throughput
```

Use sustainable throughput after accounting for downstream writes, not a broker benchmark in isolation.

## Failure modes

- New producer emits a required field before all consumers understand it.
- Replayed events resend customer email or duplicate a provider call.
- A schema registry checks syntax while a field's business meaning changes.
- Consumer deploy rolls back but the producer already emitted an incompatible version.
- Replay saturates the primary database and delays live processing.

## Interview prompts

- Which compatibility direction does a rolling deployment need?
- How would you rebuild an index without sending notifications again?
- What position pairs a snapshot with the remaining log?
- How long must old event versions remain readable?

## Two-minute answer

Treat events as long-lived APIs. Use additive changes, explicit defaults, stable event meaning, compatibility gates, and a supported-version window. Preserve occurrence time and entity versions separately. Design replay per purpose: a bounded source range, idempotent or suppressed external effects, capacity limits, progress checkpoints, and validation before cutover. Size retention and snapshots against the required recovery time and privacy policy.

