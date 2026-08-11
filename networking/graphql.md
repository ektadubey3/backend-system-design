# GraphQL

GraphQL is a query language and execution model for APIs built around a typed schema. Clients specify the shape of data they need.

Its main system-design trade-off is moving flexibility from endpoint design into **query planning, resolver execution, authorization, cost control, and caching**.

## Interview TL;DR

1. GraphQL provides a typed schema and client-shaped queries.
2. One GraphQL request can still cause many backend/database operations; “one HTTP request” does not mean “one cheap query.”
3. The N+1 resolver problem requires batching, caching, or different data-fetch planning.
4. Query depth, breadth, aliases, lists, and expensive fields require cost controls.
5. Authorization must protect the underlying business object/field—not only the top-level query.
6. GraphQL error responses may contain partial `data` plus `errors`; clients must handle partial success intentionally.
7. HTTP/CDN caching is less straightforward than resource URLs, but persisted operations and normalized client caches can help.
8. Schema evolution is often additive; removing/renaming fields needs deprecation and client migration.
9. Federation can distribute schema ownership, but creates cross-subgraph latency and operational coupling.
10. Choose GraphQL when client flexibility is worth the server-side execution complexity.

## Schema Mental Model

```graphql
type Query {
  user(id: ID!): User
}

type User {
  id: ID!
  name: String!
  orders(first: Int!, after: String): OrderConnection!
}
```

The schema is an API contract, not a database schema.

## Queries and Mutations

Queries request data.

Mutations request state-changing operations.

Do not assume mutation operations are automatically transactional across all resolver work.

The transaction boundary belongs to the backend data/service design.

## N+1

Naive resolver flow:

```text
load 100 orders
  ↓
for each order:
  load customer
```

This can create 101 queries.

Mitigations:

- batch loader/DataLoader pattern;
- join/preload;
- request-scoped cache;
- service batch API;
- materialized projection.

## Query Cost

A client may request:

```graphql
users {
  orders {
    items {
      product {
        reviews {
          author { ... }
        }
      }
    }
  }
}
```

Control:

- depth;
- list limits;
- operation complexity/cost score;
- timeouts;
- persisted/allowlisted operations for sensitive APIs;
- rate limits by estimated cost.

## Authorization

Bad model:

```text
authenticated user
→ may query any reachable field
```

Authorization should be enforced at domain boundaries.

Examples:

- user may view own order;
- support agent needs scoped role;
- sensitive fields need field/object policy.

Avoid leaking whether forbidden objects exist when that itself is sensitive.

## Partial Errors

GraphQL can return both data and errors.

Design clients and observability to distinguish:

- full success;
- partial success;
- top-level failure.

A `200` HTTP response does not necessarily mean every requested field succeeded.

## Caching

Challenges:

- one endpoint;
- many query shapes;
- authorization-sensitive results.

Options:

- client normalized cache;
- resolver/data-loader cache;
- persisted-operation cache key;
- CDN caching for carefully designed persisted GET operations;
- backend/source cache.

## Pagination

Prefer cursor-based pagination for large mutable connections.

Bound list sizes in the schema.

## Schema Evolution

Prefer additive changes.

Use deprecation before removal.

Be careful with:

- changing nullability;
- changing enum behavior;
- semantic changes hidden behind same field.

## Federation

Useful when multiple teams own parts of one graph.

Costs:

- cross-service query plans;
- tail latency;
- ownership boundaries;
- schema coordination;
- partial failure.

Do not federate only because the organization has microservices.

## GraphQL vs REST

GraphQL is attractive when:

- multiple clients need different shapes;
- connected data is central;
- product iteration benefits from schema discovery/type tooling.

REST is often simpler when:

- resource endpoints are stable;
- HTTP caching is valuable;
- query flexibility would be dangerous;
- public integrations need simple contracts.

## Common Mistakes

- “GraphQL eliminates over-fetching” while backend resolvers still fetch too much;
- one GraphQL request treated as one DB query;
- no query-cost controls;
- auth only at gateway/top-level resolver;
- unbounded nested lists;
- introducing federation before ownership demands it.

## 2-Minute Interview Answer

> “GraphQL is useful when clients need flexible connected-data shapes, but I treat resolver execution as the hard part. I batch N+1 access, cap list/depth/complexity, enforce authorization at domain/object boundaries, and monitor resolver-level latency. I use additive schema evolution and choose federation only for real ownership boundaries. If the API is simple and cacheable by resource URL, REST may be cheaper operationally.”

## Senior-Level Follow-ups

- persisted queries;
- schema registry/compatibility;
- query-plan observability;
- federation/subgraph outage;
- partial results;
- subscriptions;
- data-loader cache scope.

## References

- [GraphQL Specification](https://spec.graphql.org/)
