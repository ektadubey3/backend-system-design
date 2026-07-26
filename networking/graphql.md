# GraphQL

GraphQL is a query language for APIs and a server-side runtime for executing those queries against a typed schema.

Unlike REST, where the server defines a fixed response shape for each endpoint, GraphQL allows clients to request exactly the fields they need.

Example query:

```graphql
query GetUser {
  user(id: "user-123") {
    id
    name
    email
  }
}
```

Example response:

```json
{
  "data": {
    "user": {
      "id": "user-123",
      "name": "Alex Morgan",
      "email": "alex@example.com"
    }
  }
}
```

A GraphQL API usually exposes a single endpoint:

```http
POST /graphql
```

The endpoint accepts different operations defined by the GraphQL schema.

GraphQL is commonly used for:

* Web applications
* Mobile applications
* Backend-for-frontend systems
* Developer platforms
* E-commerce applications
* Social networks
* Analytics dashboards
* Multi-service data aggregation
* Applications with complex frontend data requirements

GraphQL is not a database, transport protocol, or replacement for backend business logic. It is an API layer that provides a typed interface over one or more data sources.

---

## Why GraphQL?

GraphQL is useful when clients need flexible access to connected data.

### 1. Clients Request Only What They Need

REST responses are usually defined by the server.

A REST endpoint may return:

```json
{
  "id": "user-123",
  "name": "Alex Morgan",
  "email": "alex@example.com",
  "phone": "+1-555-0100",
  "address": {},
  "preferences": {},
  "createdAt": "2026-07-25T10:00:00Z"
}
```

A mobile client may need only:

```graphql
query {
  user(id: "user-123") {
    id
    name
  }
}
```

This reduces unnecessary response data.

---

### 2. Reduced Over-Fetching

Over-fetching happens when an API returns more data than the client needs.

Example:

```text
Client needs:
- Product name
- Product price

Server returns:
- Product name
- Product price
- Description
- Inventory history
- Supplier details
- Reviews
- Audit metadata
```

GraphQL allows the client to choose the required fields.

---

### 3. Reduced Under-Fetching

Under-fetching happens when the client must make multiple requests to assemble one screen.

REST example:

```text
GET /users/123
GET /users/123/orders
GET /orders/789/items
GET /products/456
```

GraphQL can retrieve related data in one operation:

```graphql
query {
  user(id: "user-123") {
    id
    name
    orders {
      id
      items {
        quantity
        product {
          id
          name
          price
        }
      }
    }
  }
}
```

The GraphQL server may still call multiple backend services internally, but the client interacts with one API operation.

---

### 4. Strongly Typed Schema

GraphQL APIs are defined using a schema.

```graphql
type User {
  id: ID!
  name: String!
  email: String!
}
```

The schema describes:

* Available types
* Fields
* Relationships
* Queries
* Mutations
* Subscriptions
* Arguments
* Nullability

This improves:

* Documentation
* Validation
* Tooling
* Code generation
* Client development
* Contract clarity

---

### 5. Better Frontend Independence

Frontend teams can request new combinations of existing fields without requiring a new backend endpoint for every screen.

This can reduce coordination overhead between frontend and backend teams.

The backend still controls:

* Which fields exist
* Authorization
* Validation
* Business logic
* Data-source access
* Performance limits

---

### 6. Schema Introspection

GraphQL schemas can be inspected programmatically.

Development tools can use introspection to provide:

* Autocomplete
* Schema documentation
* Query validation
* Type generation
* Interactive API exploration

Example tools often display available operations directly from the schema.

---

### 7. API Evolution Without Frequent Versioning

GraphQL schemas commonly evolve by adding new fields.

Old clients continue requesting the fields they already understand.

Deprecated fields can remain temporarily available:

```graphql
type User {
  id: ID!
  fullName: String!
  name: String @deprecated(reason: "Use fullName instead")
}
```

This can reduce the need for versions such as:

```text
/api/v1
/api/v2
/api/v3
```

Breaking schema changes still require careful migration.

---

## Core Concepts

## 1. Schema

The schema is the contract between clients and the GraphQL server.

Example:

```graphql
type Query {
  user(id: ID!): User
  products(limit: Int, cursor: String): ProductConnection!
}

type Mutation {
  createOrder(input: CreateOrderInput!): CreateOrderPayload!
}

type User {
  id: ID!
  name: String!
  email: String!
}

type Product {
  id: ID!
  name: String!
  price: Money!
}

type Money {
  amount: Float!
  currency: String!
}
```

The schema defines what clients can request, not how the data is stored.

---

## 2. Object Types

Object types represent entities in the GraphQL domain.

```graphql
type Product {
  id: ID!
  name: String!
  description: String
  price: Money!
  category: Category!
}
```

Each field has a type.

Fields may be:

* Scalars
* Objects
* Lists
* Enums
* Interfaces
* Unions

---

## 3. Scalar Types

GraphQL includes built-in scalar types:

* `Int`
* `Float`
* `String`
* `Boolean`
* `ID`

Example:

```graphql
type User {
  id: ID!
  name: String!
  age: Int
  verified: Boolean!
}
```

Custom scalars may represent:

* Date and time
* Email address
* URL
* Decimal
* Currency
* UUID
* JSON

Example:

```graphql
scalar DateTime
scalar Decimal
```

---

## 4. Nullability

GraphQL fields are nullable by default.

```graphql
name: String
```

This field may return a string or `null`.

A non-null field uses `!`:

```graphql
name: String!
```

A list can have several nullability combinations:

```graphql
items: [Product]
items: [Product!]
items: [Product]!
items: [Product!]!
```

Their meanings differ:

| Type          | Meaning                                    |
| ------------- | ------------------------------------------ |
| `[Product]`   | List may be null, and items may be null    |
| `[Product!]`  | List may be null, but items cannot be null |
| `[Product]!`  | List cannot be null, but items may be null |
| `[Product!]!` | Neither list nor items can be null         |

Nullability is an important API contract decision.

---

## 5. Queries

Queries retrieve data.

```graphql
query GetProduct {
  product(id: "prod-123") {
    id
    name
    price {
      amount
      currency
    }
  }
}
```

Queries should generally not cause business-side effects.

They are conceptually similar to read operations in REST.

---

## 6. Mutations

Mutations change server-side state.

```graphql
mutation CreateOrder {
  createOrder(
    input: {
      customerId: "cust-123"
      items: [
        {
          productId: "prod-456"
          quantity: 2
        }
      ]
    }
  ) {
    order {
      id
      status
    }
    errors {
      code
      message
    }
  }
}
```

Mutations are used for:

* Creating resources
* Updating resources
* Deleting resources
* Triggering business workflows
* Executing commands

GraphQL mutations are executed serially within a single operation.

---

## 7. Subscriptions

Subscriptions provide server-to-client updates, commonly over a persistent connection.

Example:

```graphql
subscription OrderStatusChanged {
  orderStatusChanged(orderId: "order-789") {
    id
    status
    updatedAt
  }
}
```

Subscriptions are useful for:

* Live notifications
* Order tracking
* Chat applications
* Real-time dashboards
* Collaborative applications

They may use:

* WebSocket
* Server-sent events
* Managed pub-sub infrastructure

Subscriptions introduce additional operational complexity and should be used only when real-time updates are required.

---

## 8. Resolvers

Resolvers are functions that return data for GraphQL fields.

Example pseudocode:

```javascript
const resolvers = {
  Query: {
    user: async (_, args, context) => {
      return context.userService.getById(args.id);
    }
  },

  User: {
    orders: async (user, _, context) => {
      return context.orderService.getByUserId(user.id);
    }
  }
};
```

Resolvers may retrieve data from:

* Databases
* REST APIs
* gRPC services
* Caches
* Message systems
* External APIs
* Other GraphQL services

Resolvers should remain thin and delegate business logic to domain services.

---

## 9. Arguments

Fields can accept arguments.

```graphql
type Query {
  product(id: ID!): Product
  products(
    categoryId: ID
    limit: Int
    cursor: String
  ): ProductConnection!
}
```

Client query:

```graphql
query {
  products(categoryId: "cat-123", limit: 20) {
    edges {
      node {
        id
        name
      }
    }
  }
}
```

Arguments commonly support:

* Resource identifiers
* Filtering
* Sorting
* Pagination
* Search
* Field-specific options

---

## 10. Input Types

Input objects define structured mutation or query inputs.

```graphql
input CreateUserInput {
  name: String!
  email: String!
  password: String!
}
```

Mutation:

```graphql
type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
}
```

Input types improve consistency and allow inputs to evolve over time.

---

## 11. Enums

Enums restrict values to a predefined set.

```graphql
enum OrderStatus {
  PENDING
  CONFIRMED
  SHIPPED
  DELIVERED
  CANCELLED
}
```

Usage:

```graphql
type Order {
  id: ID!
  status: OrderStatus!
}
```

Enums improve schema clarity but must be evolved carefully because clients may not handle unknown values safely.

---

## 12. Interfaces

Interfaces define fields shared by multiple object types.

```graphql
interface Node {
  id: ID!
}

type User implements Node {
  id: ID!
  name: String!
}

type Product implements Node {
  id: ID!
  name: String!
}
```

Clients can query shared fields:

```graphql
query {
  node(id: "123") {
    id
  }
}
```

---

## 13. Unions

Unions allow a field to return one of several unrelated object types.

```graphql
union SearchResult = User | Product | Order
```

Query:

```graphql
query {
  search(text: "alex") {
    ... on User {
      id
      name
    }

    ... on Product {
      id
      name
      price {
        amount
        currency
      }
    }
  }
}
```

---

## 14. Fragments

Fragments allow clients to reuse field selections.

```graphql
fragment ProductSummary on Product {
  id
  name
  price {
    amount
    currency
  }
}
```

Usage:

```graphql
query {
  featuredProducts {
    ...ProductSummary
  }

  recommendedProducts {
    ...ProductSummary
  }
}
```

Fragments reduce duplication in client queries.

---

## 15. Aliases

Aliases allow clients to request the same field multiple times with different arguments.

```graphql
query {
  featured: products(category: "featured") {
    id
    name
  }

  discounted: products(category: "discounted") {
    id
    name
  }
}
```

The response uses the aliases:

```json
{
  "data": {
    "featured": [],
    "discounted": []
  }
}
```

---

## 16. Variables

Variables separate query structure from values.

```graphql
query GetUser($userId: ID!) {
  user(id: $userId) {
    id
    name
  }
}
```

Variables:

```json
{
  "userId": "user-123"
}
```

Variables improve:

* Query reuse
* Security
* Logging
* Client tooling
* Persisted-query support

---

## 17. Operation Names

GraphQL operations should have descriptive names.

Prefer:

```graphql
query GetCustomerOrders {
  customer(id: "cust-123") {
    orders {
      id
    }
  }
}
```

Avoid anonymous operations in production:

```graphql
query {
  customer(id: "cust-123") {
    orders {
      id
    }
  }
}
```

Operation names improve observability and debugging.

---

## 18. Response Structure

A GraphQL response may contain:

* `data`
* `errors`
* `extensions`

Example:

```json
{
  "data": {
    "user": null
  },
  "errors": [
    {
      "message": "User not found",
      "path": ["user"],
      "extensions": {
        "code": "NOT_FOUND"
      }
    }
  ]
}
```

GraphQL can return partial data when one field fails and other fields succeed.

---

## 19. Partial Failures

Consider this query:

```graphql
query {
  user(id: "user-123") {
    id
    name
    recommendations {
      id
      name
    }
  }
}
```

The user may load successfully while the recommendation service fails.

Response:

```json
{
  "data": {
    "user": {
      "id": "user-123",
      "name": "Alex Morgan",
      "recommendations": null
    }
  },
  "errors": [
    {
      "message": "Recommendation service unavailable",
      "path": ["user", "recommendations"]
    }
  ]
}
```

Clients must be prepared to handle both `data` and `errors` in the same response.

---

## 20. DataLoader and Batching

A common GraphQL performance problem is the N+1 query issue.

Suppose a query retrieves 100 orders and resolves the customer for each order.

Naive approach:

```text
1 query to fetch orders
100 queries to fetch customers

Total: 101 queries
```

Batching approach:

```text
1 query to fetch orders
1 query to fetch all required customers

Total: 2 queries
```

DataLoader-style utilities provide:

* Request-scoped batching
* Request-scoped caching
* Duplicate-key elimination

Example pseudocode:

```javascript
const customerLoader = new DataLoader(async customerIds => {
  const customers = await customerService.getByIds(customerIds);
  return customerIds.map(id =>
    customers.find(customer => customer.id === id)
  );
});
```

---

## 21. Pagination

Large GraphQL collections should be paginated.

### Offset Pagination

```graphql
query {
  products(offset: 40, limit: 20) {
    id
    name
  }
}
```

This is simple but can become inefficient and unstable when records change.

### Cursor Pagination

```graphql
query {
  products(first: 20, after: "cursor-123") {
    edges {
      cursor
      node {
        id
        name
      }
    }

    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

Cursor pagination is generally better for:

* Large datasets
* Activity feeds
* Frequently changing collections
* Infinite scrolling

---

## 22. Introspection

GraphQL introspection allows clients to query schema information.

It enables:

* API explorers
* Autocomplete
* Schema documentation
* Client generation
* Validation tooling

Example:

```graphql
query {
  __schema {
    queryType {
      name
    }
  }
}
```

Production systems may restrict introspection for untrusted users, but disabling it is not a substitute for authorization and security controls.

---

## 23. Persisted Queries

Persisted queries allow clients to send a known query identifier instead of the full query text.

Instead of:

```json
{
  "query": "query GetProduct { product(id: \"prod-123\") { id name } }"
}
```

The client may send:

```json
{
  "id": "sha256-query-hash",
  "variables": {
    "productId": "prod-123"
  }
}
```

Benefits include:

* Smaller requests
* Query allowlisting
* Improved caching
* Reduced parsing overhead
* Better control over production operations

---

## Architecture

A production GraphQL architecture may look like this:

```text
                          ┌────────────────────┐
                          │       Client       │
                          │ Web / Mobile / API │
                          └─────────┬──────────┘
                                    │
                                   HTTPS
                                    │
                          ┌─────────▼──────────┐
                          │    CDN / Edge      │
                          │ TLS / DDoS Control │
                          └─────────┬──────────┘
                                    │
                          ┌─────────▼──────────┐
                          │    API Gateway     │
                          │ Authentication     │
                          │ Rate Limiting      │
                          └─────────┬──────────┘
                                    │
                          ┌─────────▼──────────┐
                          │  GraphQL Gateway   │
                          │ Schema Validation  │
                          │ Query Planning     │
                          │ Authorization      │
                          └─────────┬──────────┘
                                    │
                    ┌───────────────┼────────────────┐
                    │               │                │
          ┌─────────▼────────┐ ┌────▼──────────┐ ┌───▼────────────┐
          │ User Service     │ │ Order Service │ │ Product Service│
          │ REST / gRPC      │ │ REST / gRPC   │ │ REST / gRPC    │
          └─────────┬────────┘ └────┬──────────┘ └───┬────────────┘
                    │               │                │
                    └───────────────┼────────────────┘
                                    │
                  ┌─────────────────┼─────────────────┐
                  │                 │                 │
          ┌───────▼──────┐  ┌──────▼──────┐  ┌────────▼────────┐
          │ Distributed  │  │ Database    │  │ Event Stream    │
          │ Cache        │  │ Cluster     │  │ Message Queue   │
          └──────────────┘  └─────────────┘  └─────────────────┘
```

### Request Flow

1. The client sends a GraphQL operation over HTTPS.
2. The edge layer handles TLS and traffic protection.
3. The API gateway authenticates and rate-limits the request.
4. The GraphQL server parses the operation.
5. The operation is validated against the schema.
6. Authorization rules are evaluated.
7. Query depth and complexity limits are checked.
8. Resolvers fetch data from backend services.
9. DataLoader batches repeated data requests.
10. Results are assembled according to the requested field selection.
11. The response returns with `data`, `errors`, or both.
12. Metrics and traces are recorded per operation and resolver.

---

## Monolithic GraphQL Architecture

A simple application may use one GraphQL server connected directly to application services.

```text
Client
  |
  v
GraphQL Server
  |
  ├── User Module
  ├── Product Module
  ├── Order Module
  └── Payment Module
        |
        v
     Database
```

This architecture is suitable for:

* Small teams
* Early-stage products
* Modular monoliths
* Applications with one deployment unit

Advantages:

* Simple deployment
* Easy local development
* Central schema management
* Lower operational complexity

Risks:

* Large schema ownership conflicts
* Tight coupling
* Slower deployments as the system grows
* Resolver logic becoming difficult to maintain

---

## GraphQL Gateway Architecture

Larger systems may place a GraphQL gateway in front of multiple services.

```text
Client
  |
  v
GraphQL Gateway
  |
  ├── User Service
  ├── Product Service
  ├── Order Service
  └── Payment Service
```

The gateway provides a unified schema while backend services retain separate responsibilities.

Benefits include:

* One client-facing API
* Central authentication
* Shared schema
* Cross-service data aggregation
* Reduced frontend orchestration

Challenges include:

* Query-planning complexity
* Cross-service latency
* Distributed ownership
* Schema coordination
* Partial failure handling

---

## Federated GraphQL Architecture

GraphQL federation allows multiple teams to own parts of a shared graph.

```text
                         Unified Graph
                              |
                      GraphQL Router
                              |
          ┌───────────────────┼───────────────────┐
          │                   │                   │
   User Subgraph       Product Subgraph     Order Subgraph
          │                   │                   │
     User Database       Product Database      Order Database
```

Each subgraph defines part of the schema.

Example:

```graphql
type User @key(fields: "id") {
  id: ID!
  name: String!
}
```

Another subgraph may extend the same entity:

```graphql
extend type User @key(fields: "id") {
  id: ID! @external
  orders: [Order!]!
}
```

Federation is useful when:

* Multiple teams own separate domains
* Services deploy independently
* A unified client graph is required
* Schema governance is well established

Federation adds significant operational and organizational complexity.

---

## Comparison

## GraphQL vs REST

| Category       | GraphQL                      | REST                         |
| -------------- | ---------------------------- | ---------------------------- |
| API model      | Typed graph                  | Resources                    |
| Endpoints      | Usually one                  | Multiple                     |
| Response shape | Client-defined               | Server-defined               |
| Over-fetching  | Reduced                      | Possible                     |
| Under-fetching | Reduced                      | Possible                     |
| Caching        | More complex                 | Native HTTP caching          |
| Error handling | Data and errors may coexist  | Status-code oriented         |
| Versioning     | Schema evolution             | Often URL or header versions |
| File uploads   | Requires conventions         | Straightforward              |
| Monitoring     | Requires operation awareness | Endpoint-based               |
| Learning curve | Higher                       | Lower                        |
| Best fit       | Complex connected data       | General-purpose APIs         |

Use GraphQL when clients require flexible access to many related resources.

Use REST when simple resource operations, HTTP caching, and operational simplicity are priorities.

---

## GraphQL vs gRPC

| Category          | GraphQL                 | gRPC                          |
| ----------------- | ----------------------- | ----------------------------- |
| Primary use       | Client-facing data APIs | Service-to-service RPC        |
| Schema            | GraphQL SDL             | Protocol Buffers              |
| Payload           | Usually JSON            | Binary                        |
| Browser support   | Strong                  | Limited without adapters      |
| Query flexibility | High                    | Fixed operations              |
| Performance       | Moderate                | High                          |
| Streaming         | Subscriptions           | Native streaming              |
| Human readability | High                    | Low                           |
| Best fit          | Web and mobile clients  | Internal distributed services |

GraphQL and gRPC can be used together.

```text
Client
  |
  | GraphQL
  v
GraphQL Gateway
  |
  | gRPC
  v
Backend Services
```

---

## GraphQL vs SOAP

| Category               | GraphQL                    | SOAP                          |
| ---------------------- | -------------------------- | ----------------------------- |
| Type                   | Query language and runtime | Protocol                      |
| Common format          | JSON                       | XML                           |
| Contract               | GraphQL schema             | WSDL                          |
| Client-selected fields | Yes                        | No                            |
| Message size           | Usually smaller            | Usually larger                |
| Enterprise standards   | Limited                    | Extensive                     |
| Tooling                | Modern API tooling         | Mature enterprise tooling     |
| Best fit               | Modern application APIs    | Formal enterprise integration |

---

## GraphQL vs WebSocket

GraphQL and WebSocket are not direct alternatives.

GraphQL defines API operations and schema behavior.

WebSocket provides persistent bidirectional transport.

GraphQL subscriptions may use WebSocket.

```text
GraphQL Subscription
        |
        v
WebSocket Transport
```

Use GraphQL queries and mutations for standard request-response interactions.

Use GraphQL subscriptions only when server-driven updates are required.

---

## GraphQL vs Backend-for-Frontend

GraphQL and backend-for-frontend are complementary.

A backend-for-frontend is a service designed for a specific client type.

```text
Web Client
    |
Web BFF
    |
Backend Services
```

```text
Mobile Client
     |
Mobile BFF
     |
Backend Services
```

A BFF may expose GraphQL to provide flexible client-specific data access.

---

## Real-World Example: E-Commerce Product Page

Consider an e-commerce product page that needs:

* Product details
* Pricing
* Inventory
* Seller information
* Reviews
* Recommendations
* Current user's cart status

### GraphQL Query

```graphql
query GetProductPage(
  $productId: ID!
  $reviewLimit: Int!
  $recommendationLimit: Int!
) {
  product(id: $productId) {
    id
    name
    description

    price {
      amount
      currency
    }

    inventory {
      available
      estimatedDelivery
    }

    seller {
      id
      name
      rating
    }

    reviews(first: $reviewLimit) {
      edges {
        node {
          id
          rating
          title
          author {
            id
            displayName
          }
        }
      }

      pageInfo {
        hasNextPage
        endCursor
      }
    }

    recommendations(first: $recommendationLimit) {
      id
      name
      price {
        amount
        currency
      }
    }

    viewerContext {
      isInCart
      isInWishlist
    }
  }
}
```

Variables:

```json
{
  "productId": "prod-123",
  "reviewLimit": 5,
  "recommendationLimit": 8
}
```

### Internal Resolver Flow

```text
GraphQL Gateway
      |
      ├── Product Resolver
      │       └── Product Service
      |
      ├── Price Resolver
      │       └── Pricing Service
      |
      ├── Inventory Resolver
      │       └── Inventory Service
      |
      ├── Seller Resolver
      │       └── Seller Service
      |
      ├── Review Resolver
      │       └── Review Service
      |
      ├── Recommendation Resolver
      │       └── Recommendation Service
      |
      └── Viewer Context Resolver
              ├── Cart Service
              └── Wishlist Service
```

### Successful Response

```json
{
  "data": {
    "product": {
      "id": "prod-123",
      "name": "Wireless Headphones",
      "description": "Noise-cancelling wireless headphones",
      "price": {
        "amount": 149.99,
        "currency": "USD"
      },
      "inventory": {
        "available": true,
        "estimatedDelivery": "2026-07-29"
      },
      "seller": {
        "id": "seller-456",
        "name": "Audio Store",
        "rating": 4.8
      },
      "reviews": {
        "edges": [],
        "pageInfo": {
          "hasNextPage": true,
          "endCursor": "review-cursor-5"
        }
      },
      "recommendations": [],
      "viewerContext": {
        "isInCart": false,
        "isInWishlist": true
      }
    }
  }
}
```

### Partial Failure

Suppose the recommendation service is unavailable.

```json
{
  "data": {
    "product": {
      "id": "prod-123",
      "name": "Wireless Headphones",
      "recommendations": null
    }
  },
  "errors": [
    {
      "message": "Recommendations are temporarily unavailable",
      "path": [
        "product",
        "recommendations"
      ],
      "extensions": {
        "code": "SERVICE_UNAVAILABLE"
      }
    }
  ]
}
```

The main product page can still render.

### N+1 Problem

Suppose the product page retrieves 20 reviews and each review requests its author.

Naive execution:

```text
1 request for reviews
20 requests for authors

Total: 21 backend calls
```

With batching:

```text
1 request for reviews
1 batched request for all authors

Total: 2 backend calls
```

### Create Order Mutation

```graphql
mutation CreateOrder($input: CreateOrderInput!) {
  createOrder(input: $input) {
    order {
      id
      status
      total {
        amount
        currency
      }
    }

    errors {
      code
      field
      message
    }
  }
}
```

Variables:

```json
{
  "input": {
    "customerId": "cust-123",
    "idempotencyKey": "checkout-session-789",
    "items": [
      {
        "productId": "prod-123",
        "quantity": 2
      }
    ]
  }
}
```

Response:

```json
{
  "data": {
    "createOrder": {
      "order": {
        "id": "order-789",
        "status": "PENDING",
        "total": {
          "amount": 299.98,
          "currency": "USD"
        }
      },
      "errors": []
    }
  }
}
```

---

## Best Practices

## 1. Design the Schema Around the Domain

Model business concepts rather than database tables.

Good:

```graphql
type Order {
  id: ID!
  customer: Customer!
  items: [OrderItem!]!
  total: Money!
  status: OrderStatus!
}
```

Avoid exposing raw persistence details:

```graphql
type OrdersTableRow {
  order_pk: Int!
  customer_fk: Int!
  status_code: Int!
}
```

The schema should remain stable even when storage changes.

---

## 2. Keep Resolvers Thin

Resolvers should coordinate data access, not contain large amounts of business logic.

Prefer:

```text
Resolver
   |
   v
Domain Service
   |
   v
Repository or External Service
```

Avoid embedding:

* Payment rules
* Inventory rules
* Authorization policies
* Complex transactions
* Workflow orchestration

directly in resolver functions.

---

## 3. Prevent the N+1 Problem

Use request-scoped batching and caching.

Batch:

* Users by ID
* Products by ID
* Orders by customer ID
* Permissions by principal
* Inventory by product ID

Monitor backend-call counts per GraphQL operation.

---

## 4. Enforce Query Depth Limits

A deeply nested query can consume excessive resources.

Example:

```graphql
query {
  users {
    orders {
      items {
        product {
          seller {
            products {
              reviews {
                author {
                  orders {
                    id
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

Set a maximum allowed depth based on expected operations.

---

## 5. Enforce Query Complexity Limits

Depth alone does not measure cost.

This shallow query may still be expensive:

```graphql
query {
  users(first: 10000) {
    id
    name
  }
}
```

Assign costs to fields and list sizes.

Example:

```text
user: 1 point
orders: 5 points
reviews: 10 points
recommendations: 20 points
```

Reject operations that exceed the allowed budget.

---

## 6. Limit Pagination Sizes

Every list field should have a maximum page size.

Example:

```graphql
type Query {
  products(first: Int = 20, after: String): ProductConnection!
}
```

Server-side rules may enforce:

```text
Default page size: 20
Maximum page size: 100
```

Never trust client-provided limits without validation.

---

## 7. Perform Field-Level Authorization

Do not authorize only at the top-level query.

A user may access the user object but not sensitive fields.

```graphql
type User {
  id: ID!
  displayName: String!
  email: String!
  salary: Decimal
}
```

Authorization may differ for:

* `displayName`
* `email`
* `salary`
* Administrative fields
* Tenant-specific fields

Authorization should be enforced close to the protected data.

---

## 8. Avoid Exposing Sensitive Fields in the Schema

Fields that should never be public should not exist in the client-facing schema.

Do not expose:

```graphql
type User {
  passwordHash: String
  internalRiskScore: Float
  databaseKey: String
}
```

Schema visibility is not an authorization mechanism.

---

## 9. Use Consistent Mutation Payloads

Prefer payload types that support future additions.

```graphql
type CreateUserPayload {
  user: User
  errors: [UserError!]!
}
```

Instead of returning the object directly:

```graphql
createUser(input: CreateUserInput!): User
```

Payload types can later include:

* Validation errors
* Operation identifiers
* Warnings
* Client mutation IDs
* Additional metadata

---

## 10. Use Structured Errors

Define stable error codes.

```json
{
  "message": "Email address is already registered",
  "extensions": {
    "code": "EMAIL_ALREADY_EXISTS",
    "field": "email"
  }
}
```

Do not require clients to parse human-readable messages.

---

## 11. Use Operation Names

Require named operations in production.

Good:

```graphql
query GetCurrentUser {
  viewer {
    id
    name
  }
}
```

Poor:

```graphql
query {
  viewer {
    id
    name
  }
}
```

Named operations improve:

* Metrics
* Logging
* Tracing
* Alerting
* Query allowlisting

---

## 12. Use Persisted Queries for Trusted Clients

Persisted queries reduce arbitrary query execution.

Benefits include:

* Smaller payloads
* Better CDN behavior
* Query allowlisting
* Easier cost analysis
* Improved operational control

They are especially useful for first-party mobile and web applications.

---

## 13. Cache at the Correct Layer

GraphQL usually uses one HTTP endpoint, which makes traditional URL-based caching less effective.

Possible caching layers include:

* Client normalized cache
* Resolver cache
* DataLoader request cache
* Distributed object cache
* Persisted-query response cache
* CDN cache for known operations
* Backend service cache

Cache based on:

* Query
* Variables
* User identity
* Authorization scope
* Locale
* Tenant

Do not share private results across users.

---

## 14. Monitor Resolver Performance

Track:

* Operation duration
* Resolver duration
* Backend calls per resolver
* Database query count
* DataLoader batch size
* Cache hit rate
* Error rate
* Query depth
* Query complexity
* Response size

A single slow resolver can delay the entire operation.

---

## 15. Apply Timeouts and Cancellation

Set timeouts for:

* Full GraphQL operations
* Individual backend calls
* Database queries
* External API calls
* Subscription connections

Cancel downstream work when the client disconnects or the request deadline expires.

---

## 16. Use Nullable Fields Carefully

Making every field non-null can cause errors to propagate higher through the response.

Example:

```graphql
type Product {
  id: ID!
  recommendations: [Product!]!
}
```

If recommendations fail, the error may nullify a larger part of the response.

Choose nullability based on whether the field is truly guaranteed.

---

## 17. Deprecate Before Removing

Use schema deprecation:

```graphql
type User {
  fullName: String!
  name: String @deprecated(reason: "Use fullName")
}
```

Monitor usage before removing deprecated fields.

Do not remove fields while active clients still depend on them.

---

## 18. Avoid Business-Specific Logic in the Client

GraphQL gives clients field-selection flexibility, but the server should still enforce business workflows.

Do not require clients to coordinate critical multi-step operations such as:

```text
1. Reserve inventory
2. Create payment
3. Create order
4. Send confirmation
```

Expose one server-controlled mutation:

```graphql
createOrder(input: CreateOrderInput!): CreateOrderPayload!
```

---

## 19. Protect Introspection and Development Tools

Interactive explorers should not be publicly exposed without appropriate controls.

Possible protections include:

* Authentication
* Environment restrictions
* Role-based access
* Network restrictions
* Production configuration

Disabling introspection alone does not secure the API.

---

## 20. Use Schema Governance

For large teams, define rules for:

* Naming
* Nullability
* Pagination
* Error types
* Ownership
* Deprecation
* Breaking changes
* Federation entities
* Security review

Automate schema checks in continuous integration.

---

## Common Mistakes

## 1. Treating GraphQL as a Database Query Language

Clients should not receive unrestricted access to database structures.

GraphQL should expose a controlled domain API.

Resolvers must enforce:

* Authorization
* Validation
* Business rules
* Query limits
* Data privacy

---

## 2. Ignoring the N+1 Problem

A query that appears simple may generate hundreds or thousands of backend calls.

Always inspect resolver behavior and use batching.

---

## 3. Allowing Unlimited Query Depth

Without depth limits, clients can submit deeply nested operations that consume excessive CPU, memory, and backend capacity.

---

## 4. Allowing Unlimited List Sizes

A client may request:

```graphql
query {
  users(first: 1000000) {
    id
  }
}
```

Always enforce server-side limits.

---

## 5. Authorizing Only the Top-Level Resolver

A user who can access an account may not be allowed to access every field on that account.

Apply field-level and resource-level authorization.

---

## 6. Returning HTTP 200 Without Clear Error Semantics

GraphQL commonly returns HTTP `200 OK` when execution produces field-level errors.

Clients must inspect the `errors` field.

Transport-level failures should still use appropriate HTTP status codes.

Document the error policy clearly.

---

## 7. Exposing Database Models Directly

This tightly couples the public API to storage design.

Database migrations may then become API-breaking changes.

Use domain-oriented schema types.

---

## 8. Making Every Field Non-Null

Overusing `!` creates brittle schemas.

One downstream failure can nullify an entire object or operation result.

Use non-null only when the value can reliably be guaranteed.

---

## 9. Using GraphQL for Every Use Case

GraphQL may be unnecessary for:

* Simple CRUD services
* File download endpoints
* Webhook receivers
* Health checks
* High-performance internal RPC
* Static resources

Use the simplest suitable interface.

---

## 10. Putting Complex Business Logic in Resolvers

Large resolvers become difficult to test and maintain.

Move logic into domain services and application workflows.

---

## 11. Ignoring Cache Design

GraphQL does not automatically solve caching.

Without deliberate caching, repeated nested queries may create high backend load.

---

## 12. Logging Full Queries With Sensitive Variables

GraphQL variables may contain:

* Passwords
* Tokens
* Personal information
* Payment details
* Search terms

Redact sensitive inputs before logging.

---

## 13. Using Subscriptions Without Operational Planning

Subscriptions create long-lived connections and require:

* Connection management
* Authentication refresh
* Pub-sub infrastructure
* Backpressure
* Reconnection logic
* Scaling strategy

Do not use subscriptions for data that can be fetched periodically.

---

## 14. Creating One Massive Schema Without Ownership

A large shared schema can become inconsistent and difficult to evolve.

Assign clear domain ownership and schema-review responsibilities.

---

## 15. Assuming One GraphQL Request Means One Backend Request

A single client operation may trigger:

* Many database queries
* Several service calls
* Cache lookups
* External API requests

Client simplicity can hide backend complexity.

Measure the complete execution plan.

---

## Interview Questions

### 1. What is GraphQL?

GraphQL is a typed query language and execution runtime that allows clients to request specific fields from an API schema.

---

### 2. What is the N+1 problem in GraphQL?

The N+1 problem occurs when resolving a list triggers one additional backend request for every item. Batching tools such as DataLoader reduce these calls.

---

### 3. How is GraphQL different from REST?

REST exposes resource-oriented endpoints with server-defined responses. GraphQL usually exposes one typed endpoint and lets clients select the response fields.

---

### 4. How do you secure a GraphQL API?

Use authentication, field-level authorization, query depth and complexity limits, pagination limits, rate limiting, input validation, safe logging, and controlled schema exposure.

---

### 5. When should GraphQL not be used?

GraphQL may not be the best choice for simple CRUD APIs, high-performance binary service communication, file transfer, or systems that do not need flexible connected-data queries.

---

## Key Takeaways

1. **GraphQL gives clients flexible access to a strongly typed data graph, reducing over-fetching and repeated client requests.**

2. **Production GraphQL systems require careful control of resolver behavior, batching, query complexity, pagination, authorization, caching, and observability.**

3. **GraphQL is most valuable when clients need complex connected data, but it should be chosen based on system requirements rather than used as a universal replacement for REST or gRPC.**
