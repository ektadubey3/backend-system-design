# Service Boundaries and Deployment Shapes

## TL;DR

Start with one deployable unit and strong internal modules unless independent scaling, security, availability, ownership, or change cadence justifies a distributed boundary. Microservices trade local code coupling for network, data, version, and operational coupling. A boundary is credible only when it owns behavior and data.

## Deployment shapes

| Shape | Strength | Cost | Good trigger |
|---|---|---|---|
| Monolith | simple calls, transactions, deployment, debugging | weak internal structure can tangle; one scaling unit | small team/domain, rapid discovery |
| Modular monolith | explicit domain APIs with local operations | module discipline and build enforcement | most growing products |
| Microservices | independent deploy/scale/failure/security ownership | partial failure, latency, schemas, distributed data, platform cost | clear autonomous capability with different operational needs |
| Cell-based architecture | repeated isolated slices limit tenant blast radius | placement/routing and capacity fragmentation | large multi-tenant systems needing containment |

“Monolith” does not imply poor design, and “microservices” does not imply loose coupling. A distributed monolith deploys many services that must change and fail together—the most expensive combination.

## Find boundaries from business capability

Look for cohesive language, rules, and data ownership: payment, inventory, identity, catalog. A service owns its schema and exposes behavior, events, and supported query views. Direct cross-service table access bypasses that contract and makes independent evolution fictional.

Avoid entity services that only mirror tables (`UserService`, `AddressService`) when every use case chatters among them. Place operations that share a transaction/invariant together. Split read models or background compute without prematurely splitting the authority.

## Coupling dimensions

- **Temporal:** must both components be available now?
- **Data:** must they agree on one schema or invariant?
- **Release:** must versions deploy together?
- **Operational:** do they share saturation and failure pools?
- **Organizational:** can one team safely own and operate the boundary?

Asynchronous messaging reduces temporal coupling but adds contract, lag, duplicate, and replay coupling. It is not automatically decoupled.

## Data and transactions

Keep hard invariants within one local transaction whenever possible. Across boundaries, use an explicit workflow with idempotency, outbox, state, and compensation. If every request needs a synchronous join across five services, revisit the boundary or build a derived read model.

## Cell-based containment

A cell contains a subset of tenants and enough compute/data dependencies to serve them. A global control plane assigns tenants; the data plane continues for existing assignments if the control plane is unavailable. Cells cap blast radius and support gradual upgrades, but need placement, capacity headroom, evacuation, and cross-cell feature policy.

## Failure modes

- Splitting by team before domain authority is understood.
- Shared database lets any service mutate any table.
- Chatty synchronous chain multiplies tail latency and availability dependency.
- One “common” service becomes a global bottleneck and release coordinator.
- Cells share a single critical cache, identity, or database and do not isolate failure.
- More services than the team can observe, secure, and operate.

## Interview prompts

- Which invariant justifies keeping components together?
- Which independent scaling or security need justifies a split?
- Who owns the data and schema migration?
- Can existing cells serve if placement control is down?

## Two-minute answer

Begin with a modular monolith organized by business capability and enforce internal APIs/data ownership. Split a service only when a stable boundary needs independent scaling, failure containment, security, ownership, or delivery cadence that outweighs network and operations cost. Keep invariants local; expose supported APIs/events and build derived reads rather than shared-table joins. For large multi-tenant blast radius, replicate the stack into cells with stable tenant placement, headroom, and a control plane that is not required for every request.

