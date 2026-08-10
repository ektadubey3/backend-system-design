# System Design Scoring Rubric

Score each category from 1 to 4.

| Dimension | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| Requirements | Jumps to solution | Basic questions | Prioritizes constraints | Reframes ambiguity and non-goals |
| Estimation | None | Performs arithmetic | Estimates design-driving values | Uses estimates to change decisions |
| Architecture | Boxes without rationale | Partial flow | Coherent end-to-end design | Clear ownership, bottlenecks, evolution |
| Data model | Generic database | Basic schema | Access-pattern-driven model | Partition/index/consistency aware |
| Scalability | “Add servers” | Basic caching/replicas | Identifies bottlenecks | Handles hot spots and 10× evolution |
| Reliability | Happy path | Generic retry | Timeouts, idempotency, degradation | Failure domains, overload, recovery |
| Consistency | Ignored | Says strong/eventual | Per-operation semantics | Conflict/order/replication reasoning |
| Security | Ignored | Mentions auth | Authorization and abuse controls | Threat boundaries and auditability |
| Observability | Ignored | Logs/metrics | SLI-oriented telemetry | Debuggability tied to failure modes |
| Trade-offs | One “best” solution | Names alternatives | Explains costs | States triggers to revisit decisions |

## Interpretation

- **10-19:** foundation gaps or unstructured answer
- **20-29:** developing system design ability
- **30-35:** strong senior-level interview signal
- **36-40:** staff-level signal when depth is consistent
