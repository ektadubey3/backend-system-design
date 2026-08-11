# PACELC

PACELC is a model for reasoning about replicated distributed data systems.

```text
If Partition:
    Availability vs Consistency
Else:
    Latency vs Consistency
```

The useful interview insight is not memorizing labels for database products. It is identifying **which operation requires which consistency/freshness guarantee and how much coordination latency you are willing to pay**.

## Interview TL;DR

1. CAP focuses on what can be guaranteed during a network partition.
2. PACELC also asks about latency/consistency when communication is healthy.
3. A nearby replica read can reduce latency but may weaken freshness.
4. Leader/quorum coordination can strengthen consistency at latency and availability cost.
5. Apply the model per operation/configuration, not as a permanent product label.
6. Translate “consistency” into a real guarantee: linearizable, read-your-writes, bounded staleness, etc.
7. Geography makes the trade-off concrete.

## Mental Model

```text
US  <----> EU <----> APAC
```

### Local read

```text
APAC replica
    ↓
low network latency
    ↓
possible staleness
```

### Coordinated read/write

```text
APAC
  ↓
leader/quorum
  ↓
more network latency
  ↓
stronger freshness/ordering
```

## During a Partition

```text
preserve invariant
      ↓
reject/delay some operations

or

continue accepting
      ↓
temporary divergence + merge/repair
```

See [CAP Theorem](cap-theorem.md).

## Outside a Partition

Ask:

- must the user read their own write?
- can a feed be seconds stale?
- can price be stale while browsing but not during checkout?
- can analytics be minutes behind?
- can writes wait for another region?

## Example

| Operation | Requirement | Possible approach |
|---|---|---|
| Browse product | bounded/eventual freshness | local replica/cache |
| Search | eventual | search projection |
| Inventory reservation | strict invariant | authoritative owner/quorum |
| Ledger mutation | strict invariant | transactional authority |
| Analytics | eventual | async pipeline |

## Multi-Region Ownership

### Single write region

Simple ordering; remote write latency and failover complexity.

### Per-key/home-region ownership

Local writes for owned keys; requires ownership routing and migration.

### Multi-writer with conflict resolution

Use only when conflicts can be safely merged or compensated.

## Consistency Vocabulary

Consider:

- linearizability;
- serializability;
- causal consistency;
- read-your-writes;
- monotonic reads;
- bounded staleness;
- eventual convergence.

## Common Mistakes

- fixed product-wide PACELC labels;
- assuming every low-latency operation can tolerate staleness;
- ignoring read-your-writes UX;
- treating regional replication as free;
- discussing consistency without an invariant or staleness bound.

## Interview Answer Template

> “PACELC matters because this is multi-region. During a partition I preserve the inventory invariant even if isolated writes fail. During healthy operation, browse/search can use local stale-tolerant projections, while reservation coordinates with the authoritative owner. The trade-off is per operation, not one label for the whole product.”

## References

- Daniel Abadi, *Consistency Tradeoffs in Modern Distributed Database System Design: CAP is Only Part of the Story*.
