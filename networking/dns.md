# DNS

DNS stands for **Domain Name System**.

It translates human-readable domain names into machine-usable network addresses.

```text
api.example.com
        |
        v
  DNS Lookup
        |
        v
  203.0.113.10
```

Without DNS, clients would need to remember and connect directly to IP addresses.

```text
Instead of:

https://203.0.113.10

Users can access:

https://api.example.com
```

DNS is a distributed, hierarchical naming system.

It is used to discover:

* Website addresses
* API endpoints
* Mail servers
* Service instances
* Load balancers
* CDN edges
* Internal services
* Verification records
* Security policies

DNS is critical to backend system design because almost every network request begins with name resolution.

```text
Client Request
      |
      v
DNS Resolution
      |
      v
Network Connection
      |
      v
Application Request
```

If DNS fails, the application may appear completely unavailable even when all backend servers are healthy.

---

## Why DNS?

DNS provides a stable naming layer between clients and changing infrastructure.

---

### 1. Human-Readable Naming

Domain names are easier to remember than IP addresses.

```text
api.example.com
```

is easier to use than:

```text
203.0.113.10
```

Names also communicate purpose.

Examples:

```text
api.example.com
auth.example.com
payments.example.com
cdn.example.com
```

---

### 2. Infrastructure Decoupling

Applications can change their server IP addresses without forcing clients to change configuration.

```text
Before:

api.example.com -> 203.0.113.10

After:

api.example.com -> 198.51.100.20
```

Clients continue using the same hostname.

This allows operators to:

* Move services
* Replace servers
* Migrate clouds
* Rotate load balancers
* Change regions
* Perform failover

---

### 3. Distributed Scalability

DNS is globally distributed.

A query does not depend on one central server.

```text
Client
  |
  v
Recursive Resolver
  |
  v
Root Name Server
  |
  v
TLD Name Server
  |
  v
Authoritative Name Server
```

This architecture supports internet-scale lookup traffic.

---

### 4. Caching

DNS responses can be cached at multiple levels.

```text
Browser Cache
      |
      v
Operating System Cache
      |
      v
Recursive Resolver Cache
      |
      v
Authoritative DNS
```

Caching reduces:

* Lookup latency
* Authoritative server load
* Network traffic
* Dependency pressure

---

### 5. Traffic Distribution

DNS can return multiple IP addresses.

```text
api.example.com

203.0.113.10
203.0.113.11
203.0.113.12
```

This can support:

* Basic load distribution
* Multi-region routing
* Failover
* Geographic routing
* Latency-based routing
* Weighted deployments

DNS is not a complete replacement for a load balancer, but it is often part of the traffic-management architecture.

---

### 6. Service Discovery

Internal services can use DNS to locate other services.

```text
orders.internal.example.com
        |
        v
10.0.2.15
```

In container platforms, DNS may resolve a service name to:

* A virtual service IP
* One or more pod addresses
* A headless service
* A service mesh endpoint

---

### 7. High Availability

DNS records can direct users away from unhealthy regions or endpoints.

```text
Primary Region
203.0.113.10

Backup Region
198.51.100.20
```

During failover:

```text
api.example.com

Old:
203.0.113.10

New:
198.51.100.20
```

The effectiveness of DNS failover depends heavily on caching and TTL settings.

---

## Core Concepts

## 1. Domain Name Hierarchy

DNS names are hierarchical.

Example:

```text
api.eu.example.com
```

Hierarchy:

```text
.                  Root
└── com            Top-Level Domain
    └── example    Registered Domain
        └── eu     Subdomain
            └── api Hostname
```

The complete domain name can be represented as:

```text
api.eu.example.com.
```

The trailing dot represents the DNS root.

It is often omitted in normal usage.

---

## 2. DNS Resolver

A resolver is responsible for finding DNS answers.

There are two common resolver roles:

* Stub resolver
* Recursive resolver

### Stub Resolver

The stub resolver runs on the client device or operating system.

```text
Application
    |
    v
Stub Resolver
    |
    v
Configured DNS Server
```

It usually forwards queries rather than performing the full lookup itself.

### Recursive Resolver

A recursive resolver performs the lookup on behalf of the client.

Examples include resolvers operated by:

* Internet service providers
* Cloud platforms
* Enterprises
* Public DNS providers

---

## 3. Recursive Query

In a recursive query, the client asks the resolver to return the final answer.

```text
Client:
What is the IP for api.example.com?

Resolver:
I will find the final answer.
```

Flow:

```text
Client
  |
  | Recursive Query
  v
Recursive Resolver
  |
  | Performs additional queries
  v
Final Answer
```

---

## 4. Iterative Query

In an iterative query, a DNS server returns the best information it currently has.

It may return a referral to another DNS server.

```text
Resolver:
Where is api.example.com?

Root Server:
Ask the .com name servers.
```

```text
Resolver:
Where is api.example.com?

.com Server:
Ask the example.com name servers.
```

---

## 5. Root Name Servers

Root name servers are at the top of the DNS hierarchy.

They do not normally return the IP address for a typical application hostname.

Instead, they direct resolvers to the appropriate top-level-domain name servers.

```text
Query:
api.example.com

Root Response:
Ask the .com name servers
```

Root servers are implemented through many globally distributed instances.

---

## 6. Top-Level Domain Servers

Top-level-domain servers manage DNS information for domains under extensions such as:

* `.com`
* `.org`
* `.net`
* `.io`
* Country-code domains

For example:

```text
.com Name Server

example.com -> Authoritative Name Servers
```

The TLD server returns a referral to the authoritative servers for the registered domain.

---

## 7. Authoritative Name Server

An authoritative name server holds the official DNS records for a zone.

```text
Zone:
example.com

Records:
api.example.com -> 203.0.113.10
mail.example.com -> mail-provider.example.net
```

The authoritative server provides the final answer for records it controls.

It does not need to recursively resolve its own zone data.

---

## 8. DNS Resolution Flow

A complete lookup may look like this:

```text
Client
  |
  | 1. api.example.com?
  v
Recursive Resolver
  |
  | 2. Ask Root
  v
Root Name Server
  |
  | 3. Ask .com TLD
  v
.com Name Server
  |
  | 4. Ask example.com authority
  v
Authoritative Name Server
  |
  | 5. api.example.com = 203.0.113.10
  v
Recursive Resolver
  |
  | 6. Cached Answer
  v
Client
```

The resolver caches referrals and final answers to reduce future lookup work.

---

## 9. DNS Zones

A zone is an administrative portion of the DNS namespace.

Example:

```text
example.com zone
```

It may contain:

```text
example.com
www.example.com
api.example.com
mail.example.com
```

A subdomain may be delegated to a different zone.

```text
example.com
    |
    └── internal.example.com
            |
            └── Managed by different authoritative servers
```

A domain and a zone are related but not always identical.

---

## 10. Zone Delegation

Delegation assigns authority for a subdomain to another set of name servers.

Example:

```text
example.com zone

Delegates:

payments.example.com
```

The parent zone contains name-server records for the delegated child zone.

```text
payments.example.com NS ns1.payment-dns.example
payments.example.com NS ns2.payment-dns.example
```

This allows separate teams or providers to manage different parts of a namespace.

---

## 11. A Record

An `A` record maps a hostname to an IPv4 address.

```text
api.example.com. 300 IN A 203.0.113.10
```

Meaning:

```text
api.example.com -> 203.0.113.10
```

---

## 12. AAAA Record

An `AAAA` record maps a hostname to an IPv6 address.

```text
api.example.com. 300 IN AAAA 2001:db8::10
```

A hostname may have both `A` and `AAAA` records.

```text
api.example.com

A:
203.0.113.10

AAAA:
2001:db8::10
```

Clients may choose between IPv4 and IPv6 based on network support and connection behavior.

---

## 13. CNAME Record

A `CNAME` record makes one hostname an alias of another hostname.

```text
www.example.com. IN CNAME edge.cdn-provider.example.
```

Resolution:

```text
www.example.com
       |
       v
edge.cdn-provider.example
       |
       v
203.0.113.20
```

A CNAME points to a name, not directly to an IP address.

A hostname with a CNAME generally should not also contain unrelated record types at the same name.

---

## 14. MX Record

An `MX` record identifies mail servers for a domain.

```text
example.com. IN MX 10 mail1.example.com.
example.com. IN MX 20 mail2.example.com.
```

Lower preference values are normally attempted first.

```text
Priority 10 -> Primary
Priority 20 -> Backup
```

The mail-server hostnames must resolve to addresses.

---

## 15. NS Record

An `NS` record identifies authoritative name servers for a zone.

```text
example.com. IN NS ns1.dns-provider.example.
example.com. IN NS ns2.dns-provider.example.
```

Multiple name servers provide redundancy.

---

## 16. TXT Record

A `TXT` record stores arbitrary text.

Common uses include:

* Domain verification
* SPF
* DKIM
* DMARC
* Service ownership proof
* Security policies

Example:

```text
example.com. IN TXT "service-verification=abc123"
```

TXT records are often used by cloud and SaaS providers to verify domain ownership.

---

## 17. SRV Record

An `SRV` record identifies the hostname and port for a service.

Example:

```text
_service._proto.example.com
```

Conceptual record:

```text
_chat._tcp.example.com. IN SRV 10 5 5222 chat1.example.com.
```

Fields include:

* Priority
* Weight
* Port
* Target

SRV records can support service discovery where clients understand the record format.

---

## 18. PTR Record

A `PTR` record maps an IP address back to a hostname.

This is called reverse DNS.

```text
203.0.113.10
        |
        v
api.example.com
```

Reverse DNS uses special zones such as:

```text
in-addr.arpa
ip6.arpa
```

PTR records are commonly important for mail infrastructure, diagnostics, and network operations.

---

## 19. SOA Record

The Start of Authority record contains administrative information about a DNS zone.

It commonly includes:

* Primary authoritative server
* Administrative contact
* Zone serial number
* Refresh interval
* Retry interval
* Expiration interval
* Negative caching value

Conceptual example:

```text
example.com. IN SOA ns1.example.net. admin.example.com. (
  2026072801
  3600
  600
  1209600
  300
)
```

The serial number helps secondary servers detect zone changes.

---

## 20. TTL

TTL stands for **Time to Live**.

It tells resolvers how long they may cache a DNS response.

```text
api.example.com. 300 IN A 203.0.113.10
```

Here:

```text
TTL = 300 seconds
```

The record may be cached for up to five minutes.

### Low TTL

Advantages:

* Faster DNS changes
* Faster failover
* Easier migrations

Disadvantages:

* More DNS queries
* Higher resolver and authoritative load
* Less caching efficiency

### High TTL

Advantages:

* Better caching
* Lower DNS load
* Faster repeated lookups

Disadvantages:

* Slower propagation of changes
* Longer recovery from incorrect records
* Slower traffic migration

TTL selection is a trade-off.

---

## 21. Positive Caching

Positive caching stores successful DNS answers.

```text
api.example.com -> 203.0.113.10
```

If cached, future queries can be answered immediately.

```text
Client
  |
  v
Resolver Cache Hit
  |
  v
203.0.113.10
```

---

## 22. Negative Caching

Negative caching stores responses indicating that a name or record does not exist.

Example:

```text
missing.example.com -> NXDOMAIN
```

Negative caching prevents repeated queries for nonexistent names.

It also means that newly created records may remain unavailable temporarily if a negative answer was previously cached.

---

## 23. NXDOMAIN

`NXDOMAIN` means that the queried domain name does not exist.

```text
Query:
unknown.example.com

Response:
NXDOMAIN
```

This differs from a valid name that lacks the requested record type.

Example:

```text
Name exists:
example.com

Requested:
AAAA

Result:
No AAAA record
```

This may return an empty answer rather than NXDOMAIN.

---

## 24. DNS Record Set

Multiple records of the same name and type form a record set.

Example:

```text
api.example.com. IN A 203.0.113.10
api.example.com. IN A 203.0.113.11
api.example.com. IN A 203.0.113.12
```

The resolver may return all addresses.

Clients may:

* Try one address
* Rotate addresses
* Use connection racing
* Apply their own selection logic

DNS does not guarantee even traffic distribution.

---

## 25. Round-Robin DNS

Round-robin DNS changes or rotates the order of returned addresses.

```text
Query 1:
203.0.113.10
203.0.113.11
203.0.113.12
```

```text
Query 2:
203.0.113.11
203.0.113.12
203.0.113.10
```

Limitations include:

* Client-side caching
* Resolver-side caching
* Unequal backend capacity
* Long-lived connections
* Lack of immediate health awareness
* Clients choosing the first result differently

Round-robin DNS is simple traffic distribution, not precise load balancing.

---

## 26. Split-Horizon DNS

Split-horizon DNS returns different answers depending on where the query originates.

Example:

```text
Internal Client:
api.example.com -> 10.0.1.20
```

```text
External Client:
api.example.com -> 203.0.113.20
```

This is useful for:

* Private services
* Internal routing
* Hybrid cloud
* Security boundaries
* Reducing public network traversal

Split-horizon designs require careful operational management to avoid inconsistent behavior.

---

## 27. Public and Private DNS

### Public DNS

Accessible through the public internet.

Used for:

* Websites
* Public APIs
* Email
* CDN endpoints

### Private DNS

Available only within:

* Virtual private clouds
* Corporate networks
* Private data centers
* VPN-connected environments

Example:

```text
payments.service.internal
```

Private DNS should not expose internal infrastructure names publicly.

---

## 28. DNS over UDP

Most traditional DNS queries use UDP.

```text
Client
   |
   | UDP Port 53
   v
DNS Resolver
```

UDP is efficient for small request-response exchanges.

If the response is too large or truncated, the client may retry using TCP.

---

## 29. DNS over TCP

DNS uses TCP for situations such as:

* Large responses
* Zone transfers
* Truncated UDP responses
* Some DNSSEC responses
* Environments where UDP is blocked

```text
Client
   |
   | TCP Port 53
   v
DNS Server
```

DNS infrastructure should generally support both UDP and TCP on port 53.

---

## 30. EDNS

Extension Mechanisms for DNS allow clients and servers to advertise support for larger UDP responses and additional features.

Without extensions, classic DNS UDP messages have stricter size constraints.

EDNS helps support:

* Larger responses
* DNSSEC
* Extended response information
* Additional signaling

Large UDP responses still risk fragmentation and packet loss.

---

## 31. DNSSEC

DNSSEC adds cryptographic authenticity and integrity to DNS data.

It helps protect against forged DNS responses.

DNSSEC uses record types such as:

* `DNSKEY`
* `DS`
* `RRSIG`
* `NSEC`
* `NSEC3`

Validation chain:

```text
Root Trust Anchor
        |
        v
TLD Signature
        |
        v
Domain Signature
        |
        v
Validated DNS Record
```

DNSSEC proves that a response was signed by the appropriate zone.

It does not encrypt DNS queries or hide domain names.

---

## 32. DNS over HTTPS

DNS over HTTPS, or DoH, sends DNS queries over HTTPS.

```text
DNS Query
    |
    v
  HTTPS
    |
    v
TCP or QUIC
```

Benefits may include:

* Encryption between client and resolver
* Resistance to simple network interception
* Use of common HTTPS infrastructure

Operational concerns include:

* Reduced enterprise visibility
* Resolver centralization
* Policy enforcement complexity
* Application-specific resolver behavior

---

## 33. DNS over TLS

DNS over TLS, or DoT, encrypts DNS traffic using TLS.

```text
Client
   |
   | TLS
   v
DNS Resolver
```

DoT commonly uses a dedicated port.

Unlike DNSSEC, DoT protects the communication channel between the client and resolver rather than signing authoritative DNS data.

---

## 34. Anycast

Anycast allows multiple servers in different locations to advertise the same IP address.

```text
               Same DNS IP Address

Client A ─────> DNS Server Region A
Client B ─────> DNS Server Region B
Client C ─────> DNS Server Region C
```

Network routing directs clients toward a nearby or reachable instance.

Anycast is widely used for:

* Recursive resolvers
* Root servers
* Authoritative DNS
* DDoS resilience
* Global low-latency infrastructure

---

## 35. Glue Records

Glue records provide the IP addresses of name servers when those addresses are required to complete delegation.

Example:

```text
example.com NS ns1.example.com
```

To find `ns1.example.com`, the resolver may need information from the `example.com` zone, creating a circular dependency.

The parent zone can provide glue:

```text
ns1.example.com -> 203.0.113.53
```

Glue should be included only when necessary for delegation.

---

## 36. DNS Propagation

DNS changes do not instantly replace every cached value.

Propagation depends on:

* Previous TTL
* Resolver caching
* Browser caching
* Operating-system caching
* Negative caching
* Provider update timing
* Client behavior

Suppose a record has a TTL of one hour.

```text
Old record cached at 10:00
Record changed at 10:10
Cache may retain old value until 11:00
```

DNS propagation is largely cache expiration, not a single global synchronization event.

---

## Architecture

A production DNS architecture may include multiple layers.

```text
                         ┌──────────────────────┐
                         │   Browser / Client   │
                         └──────────┬───────────┘
                                    │
                         Local DNS Resolution
                                    │
                         ┌──────────▼───────────┐
                         │ Browser / OS Cache   │
                         └──────────┬───────────┘
                                    │
                         Recursive DNS Query
                                    │
                         ┌──────────▼───────────┐
                         │ Recursive Resolver   │
                         │ Cache + DNSSEC Check │
                         └──────────┬───────────┘
                                    │
               ┌────────────────────┼────────────────────┐
               │                    │                    │
      ┌────────▼────────┐  ┌────────▼────────┐  ┌────────▼────────┐
      │ Root DNS        │  │ TLD DNS         │  │ Authoritative   │
      │ Servers         │  │ Servers         │  │ DNS Provider    │
      └─────────────────┘  └─────────────────┘  └────────┬────────┘
                                                         │
                                      ┌──────────────────┼──────────────────┐
                                      │                  │                  │
                              ┌───────▼───────┐  ┌───────▼───────┐  ┌───────▼─────┐
                              │ Global Region │  │ Global Region │  │ CDN / Edge  │
                              │ A             │  │ B             │  │ Endpoints   │
                              └───────────────┘  └───────────────┘  └─────────────┘
```

---

## Resolution Request Flow

1. The application requests a hostname.
2. The browser checks its DNS cache.
3. The operating system checks its cache and local host configuration.
4. The stub resolver sends a query to the recursive resolver.
5. The recursive resolver checks its cache.
6. On a cache miss, it queries root servers.
7. The root server returns a TLD referral.
8. The resolver queries the TLD server.
9. The TLD server returns the authoritative name-server referral.
10. The resolver queries the authoritative server.
11. The authoritative server returns the record.
12. The recursive resolver caches the response.
13. The result is returned to the client.
14. The application connects to the returned address.

---

## Multi-Region DNS Architecture

DNS is commonly used to route clients to regional endpoints.

```text
                             api.example.com
                                    |
                                    v
                         Global Authoritative DNS
                                    |
               ┌────────────────────┼────────────────────┐
               │                    │                    │
        North America          Europe Region          Asia Region
        203.0.113.10           198.51.100.20          192.0.2.30
               │                    │                    │
       Regional Load            Regional Load         Regional Load
         Balancer                 Balancer              Balancer
```

Routing policies may consider:

* Geographic location
* Network latency
* Endpoint health
* Traffic weight
* Regulatory boundaries
* Region capacity

---

## DNS Failover Architecture

```text
                       api.example.com
                              |
                              v
                    Health-Aware DNS Policy
                              |
              ┌───────────────┴───────────────┐
              │                               │
      Primary Region                    Backup Region
      203.0.113.10                      198.51.100.20
              │                               │
         Healthy? Yes                      Standby
```

After failure detection:

```text
                       api.example.com
                              |
                              v
                    Health-Aware DNS Policy
                              |
                              v
                     Backup Region Address
                     198.51.100.20
```

Limitations:

* Cached old responses may remain active.
* Existing TCP connections do not automatically migrate.
* Resolver behavior varies.
* Health checks may not represent full application health.

DNS failover should be combined with application and load-balancer resilience.

---

## Internal Service Discovery Architecture

```text
                      Order Service
                            |
                            | Resolve
                            v
                 inventory.service.internal
                            |
                            v
                    Private DNS Resolver
                            |
                            v
          ┌─────────────────┼─────────────────┐
          │                 │                 │
      10.0.2.10         10.0.2.11         10.0.2.12
          │                 │                 │
     Inventory A       Inventory B       Inventory C
```

Clients may use:

* One returned address
* All returned addresses
* Client-side load balancing
* A service virtual IP
* Service-mesh discovery

---

## Comparison

## DNS vs Load Balancer

| Category            | DNS                          | Load Balancer                  |
| ------------------- | ---------------------------- | ------------------------------ |
| Primary role        | Name resolution              | Traffic distribution           |
| Decision timing     | Before connection            | During connection or request   |
| Client caching      | Common                       | Usually not visible to client  |
| Health response     | Delayed by TTL               | Often immediate                |
| Traffic granularity | Resolver or client level     | Connection or request level    |
| Session awareness   | Limited                      | Often supported                |
| Failover speed      | Cache dependent              | Usually faster                 |
| Geographic routing  | Strong                       | Usually regional unless global |
| Backend visibility  | Indirect                     | Direct                         |
| Best use            | Global discovery and routing | Precise traffic management     |

DNS and load balancers are commonly used together.

```text
Client
  |
  v
 DNS
  |
  v
Regional Load Balancer
  |
  v
Backend Instance
```

---

## DNS vs Service Registry

| Category          | DNS                      | Service Registry        |
| ----------------- | ------------------------ | ----------------------- |
| Interface         | Standard DNS queries     | Custom API or protocol  |
| Client support    | Universal                | Requires integration    |
| Metadata richness | Limited                  | Often extensive         |
| Health awareness  | Usually delayed          | Often near real time    |
| Discovery scope   | Public and private       | Usually internal        |
| Caching           | Built into DNS ecosystem | Implementation specific |
| Port discovery    | SRV records possible     | Usually native          |
| Best use          | Broad compatibility      | Dynamic microservices   |

A service registry may publish data into DNS to combine both approaches.

---

## DNS vs Hosts File

| Category        | DNS                         | Hosts File                 |
| --------------- | --------------------------- | -------------------------- |
| Management      | Centralized and distributed | Per machine                |
| Scalability     | Internet scale              | Very limited               |
| Dynamic changes | Supported                   | Manual                     |
| Caching         | Built in                    | Local static lookup        |
| Best use        | Production naming           | Local testing or overrides |

Example hosts entry:

```text
127.0.0.1 api.local
```

Hosts files are useful for development but are difficult to manage at scale.

---

## Recursive DNS vs Authoritative DNS

| Category              | Recursive Resolver         | Authoritative Server      |
| --------------------- | -------------------------- | ------------------------- |
| Role                  | Finds answers for clients  | Stores official zone data |
| Caching               | Yes                        | Not its primary purpose   |
| Queries other servers | Yes                        | Usually no recursion      |
| Used by               | End users and applications | Domain owners             |
| Returns               | Cached or resolved answer  | Authoritative zone answer |

---

## DNS over UDP vs DNS over TCP

| Category           | DNS over UDP               | DNS over TCP                 |
| ------------------ | -------------------------- | ---------------------------- |
| Connection setup   | None                       | Required                     |
| Overhead           | Lower                      | Higher                       |
| Common queries     | Most standard lookups      | Large or truncated responses |
| Reliability        | Datagram based             | Reliable byte stream         |
| Zone transfers     | No                         | Yes                          |
| Fragmentation risk | Higher for large responses | Lower at DNS message level   |
| Port               | 53                         | 53                           |

Production DNS servers should normally support both.

---

## DNSSEC vs DoH and DoT

| Category                           | DNSSEC                     | DoH / DoT                  |
| ---------------------------------- | -------------------------- | -------------------------- |
| Protects                           | DNS data authenticity      | Client-to-resolver channel |
| Encrypts query name                | No                         | Yes, on the protected hop  |
| Prevents forged authoritative data | Yes, when validated        | Not by itself              |
| Requires signed zones              | Yes                        | No                         |
| Main goal                          | Integrity and authenticity | Privacy in transit         |

They solve different security problems and can be used together.

---

## A Record vs CNAME

| Category            | A Record               | CNAME Record              |
| ------------------- | ---------------------- | ------------------------- |
| Points to           | IPv4 address           | Another hostname          |
| Extra lookup        | Usually no             | Usually yes               |
| Useful for          | Direct address mapping | Aliasing managed services |
| Root-domain support | Common                 | Traditionally restricted  |
| Dependency          | IP address             | Target hostname           |

Some DNS providers offer proprietary flattening or alias record types for root domains.

---

## Real-World Example: Global E-Commerce API

Consider a global e-commerce platform with users in multiple regions.

Requirements:

* Low-latency API access
* Regional redundancy
* Zero-downtime deployments
* Disaster recovery
* CDN integration
* Private service discovery
* Secure domain management

---

### Public DNS Design

```text
api.shop.example.com
```

The authoritative DNS provider returns the nearest healthy regional endpoint.

```text
North American Client:
api.shop.example.com -> 203.0.113.10

European Client:
api.shop.example.com -> 198.51.100.20

Asian Client:
api.shop.example.com -> 192.0.2.30
```

---

### Architecture

```text
                             User
                              |
                              v
                    Recursive DNS Resolver
                              |
                              v
                   Global Authoritative DNS
                              |
          ┌───────────────────┼───────────────────┐
          │                   │                   │
      Americas             Europe              Asia
  api-us.example.net  api-eu.example.net  api-ap.example.net
          │                   │                   │
          v                   v                   v
   Regional Load       Regional Load       Regional Load
      Balancer             Balancer             Balancer
          │                   │                   │
     API Cluster         API Cluster         API Cluster
```

---

### Example DNS Records

```text
api.shop.example.com. 60 IN CNAME global-api.dns-provider.example.
```

The DNS provider may return regional addresses based on routing policy.

```text
global-api.dns-provider.example. 60 IN A 203.0.113.10
global-api.dns-provider.example. 60 IN A 198.51.100.20
global-api.dns-provider.example. 60 IN A 192.0.2.30
```

The exact response may vary by client location and provider configuration.

---

### Normal Request Flow

```text
1. User requests api.shop.example.com
2. Recursive resolver obtains the regional answer
3. Resolver caches the answer for the TTL
4. Client connects to the regional load balancer
5. Load balancer selects a healthy API instance
6. API processes the request
```

---

### Regional Failure

Suppose the European region becomes unavailable.

```text
Before:

European Clients
      |
      v
198.51.100.20
```

Health monitoring detects failure.

The DNS policy stops returning the European endpoint.

```text
After:

European Clients
      |
      v
203.0.113.10 or 192.0.2.30
```

However, some users may continue using the failed address until cached responses expire.

For this reason:

* TTL was reduced before risky migrations.
* Regional load balancers have local failover.
* Clients use bounded connection timeouts.
* Applications retry idempotent requests.
* Critical state is replicated across regions.

DNS failover is only one layer of resilience.

---

### Blue-Green Deployment

Suppose the platform deploys a new API version.

```text
Blue Environment:
203.0.113.10

Green Environment:
203.0.113.20
```

A weighted DNS policy may gradually shift traffic.

```text
Stage 1:
Blue 95%
Green 5%

Stage 2:
Blue 75%
Green 25%

Stage 3:
Blue 50%
Green 50%

Stage 4:
Blue 0%
Green 100%
```

Limitations:

* Resolver caching reduces precise percentage control.
* Long-lived connections remain on old endpoints.
* Clients may reuse cached addresses.

For precise canary routing, use DNS to reach a load balancer and perform request-level weighting there.

---

### Private Service Discovery

Inside the regional cluster:

```text
orders.service.internal
inventory.service.internal
payments.service.internal
```

Example lookup:

```text
inventory.service.internal

10.0.4.10
10.0.4.11
10.0.4.12
```

The Order Service resolves the Inventory Service using private DNS.

```text
Order Service
      |
      v
Private DNS
      |
      v
Inventory Service Endpoints
```

The client should not assume DNS records remain unchanged forever.

It should:

* Respect TTLs
* Refresh addresses
* Retry another endpoint when safe
* Use connection pooling carefully
* Respond to service changes

---

### CDN Integration

Static content uses a CDN hostname.

```text
assets.shop.example.com
        |
        v
cdn-provider.example.net
```

DNS record:

```text
assets.shop.example.com. IN CNAME cdn-provider.example.net.
```

The CDN provider returns a nearby edge address.

```text
User
  |
  v
 DNS
  |
  v
Nearest CDN Edge
  |
  v
Cached Asset or Origin Fetch
```

---

### Email Records

The platform's domain also includes mail and security records.

```text
shop.example.com. IN MX 10 mail-provider.example.
shop.example.com. IN TXT "v=spf1 include:mail-provider.example -all"
```

Additional TXT records may support:

* DKIM
* DMARC
* Domain verification

DNS configuration affects much more than web traffic.

---

## Best Practices

## 1. Use Multiple Authoritative Name Servers

Configure at least two authoritative servers.

```text
example.com NS ns1.dns-provider.example
example.com NS ns2.dns-provider.example
```

Prefer infrastructure distributed across:

* Different availability zones
* Different regions
* Independent networks
* Multiple providers for highly critical domains

Redundancy should avoid shared failure points.

---

## 2. Select TTLs Deliberately

Do not set every record to the same TTL.

Use lower TTLs for:

* Active migrations
* Failover records
* Dynamic endpoints
* Planned cutovers

Use higher TTLs for:

* Stable records
* Verification records
* Long-lived infrastructure
* Static mail configuration

Balance agility against caching efficiency.

---

## 3. Lower TTL Before Planned Changes

Suppose a record currently has:

```text
TTL = 86400 seconds
```

Changing it immediately to a new IP does not remove previously cached answers.

A safer migration:

```text
Several TTL periods before migration:
Reduce TTL to 300 seconds

Wait for old caches to expire

Perform the record change

After stability:
Increase TTL again
```

Plan according to the previous TTL, not only the new TTL.

---

## 4. Keep DNS Records Under Version Control

Manage DNS through infrastructure as code.

Benefits include:

* Change review
* Audit history
* Automated validation
* Repeatable environments
* Safer rollback
* Reduced manual errors

Avoid untracked production changes through provider dashboards.

---

## 5. Use Automated Validation

Validate records before deployment.

Check for:

* Invalid hostnames
* Missing targets
* CNAME conflicts
* Incorrect TTLs
* Broken delegations
* Missing mail records
* Invalid DNSSEC configuration
* Accidental public exposure

Run DNS tests in continuous integration where practical.

---

## 6. Monitor Authoritative DNS

Track:

* Query volume
* Response latency
* Error rate
* `SERVFAIL`
* `NXDOMAIN`
* Timeout rate
* TCP fallback rate
* DNSSEC validation failures
* Record changes
* Provider health

DNS must be treated as production infrastructure.

---

## 7. Monitor Resolution From Multiple Regions

A DNS record may work from one network and fail from another.

Test from:

* Multiple countries
* Multiple cloud providers
* IPv4 and IPv6 networks
* Internal and external networks
* Different recursive resolvers

This helps detect:

* Routing problems
* Delegation issues
* Regional outages
* Split-horizon errors
* Propagation differences

---

## 8. Support Both UDP and TCP on Port 53

Blocking TCP port 53 can break:

* Large responses
* DNSSEC
* Zone transfers
* Truncated-query fallback

DNS is not UDP-only.

---

## 9. Use DNSSEC Where Appropriate

DNSSEC can protect against forged DNS data.

Operational requirements include:

* Correct key management
* Proper delegation
* Key rotation
* Monitoring
* Safe rollover procedures

Incorrect DNSSEC configuration can make a domain unreachable for validating resolvers.

Deploy it with automation and testing.

---

## 10. Protect Registrar and DNS Accounts

The domain registrar and DNS provider are high-value targets.

Use:

* Multi-factor authentication
* Hardware security keys
* Role-based access
* Least privilege
* Audit logging
* Change approval
* Domain lock features
* Recovery procedures

An attacker controlling DNS can redirect users even when application servers remain secure.

---

## 11. Separate Public and Private DNS

Use separate zones or controlled split-horizon DNS for internal names.

Avoid exposing:

* Internal hostnames
* Private IP addresses
* Environment names
* Infrastructure topology
* Administrative endpoints

Private services should resolve only within authorized networks.

---

## 12. Avoid Embedding Raw IP Addresses in Applications

Use stable service names.

Poor:

```text
https://203.0.113.10
```

Better:

```text
https://api.example.com
```

Hostnames make infrastructure migration and failover easier.

---

## 13. Do Not Treat DNS as Precise Load Balancing

DNS does not control every request.

Clients and resolvers cache responses.

For precise routing, combine DNS with:

* Layer 4 load balancers
* Layer 7 load balancers
* API gateways
* Service meshes
* Client-side load balancing

---

## 14. Use Health Checks Carefully

A DNS endpoint should be considered healthy only when it can serve meaningful traffic.

Check more than process availability.

Possible checks include:

* Network reachability
* Load balancer health
* Application readiness
* Critical dependency state
* Regional capacity

Avoid health checks that remove all regions because of one shared dependency failure.

---

## 15. Plan for Cache Inconsistency

During DNS changes, different clients may receive different answers.

Applications should tolerate a transition period.

Use:

* Backward-compatible deployments
* Overlapping old and new infrastructure
* Graceful connection draining
* Data compatibility
* Idempotent retries
* Monitoring of both endpoints

Do not shut down the old endpoint immediately after changing DNS.

---

## 16. Keep Old Infrastructure Available During Migration

A safe sequence is:

```text
1. Deploy new infrastructure
2. Validate it
3. Change DNS
4. Wait for old TTLs to expire
5. Monitor remaining traffic
6. Remove old infrastructure
```

This reduces downtime caused by cached records.

---

## 17. Use Shorter Names for Frequently Queried Internal Services

Extremely long names increase DNS packet size and operational complexity.

Use consistent naming such as:

```text
service.environment.region.internal
```

Example:

```text
orders.prod.eu.internal
```

Keep names descriptive but manageable.

---

## 18. Define Naming Conventions

Establish consistent patterns for:

* Environments
* Regions
* Services
* Internal zones
* Public APIs
* Administrative endpoints

Example:

```text
api.example.com
api-staging.example.com
orders.prod.us.internal
orders.prod.eu.internal
```

Consistency improves automation and troubleshooting.

---

## 19. Use CNAMEs Carefully

CNAME chains increase lookup work.

Poor:

```text
api.example.com
  -> service.example.net
  -> routing.example.org
  -> final.example.io
  -> IP address
```

Prefer short chains.

Also verify that the target remains under operational control and is not deleted unexpectedly.

---

## 20. Prevent Dangling DNS Records

A dangling record points to a resource that no longer exists.

Example:

```text
old-app.example.com
      |
      v
Deleted cloud resource
```

An attacker may be able to claim the abandoned target and serve content through the trusted hostname.

Regularly audit:

* CNAME targets
* Cloud load balancers
* Object-storage websites
* CDN distributions
* SaaS integrations
* Verification records

---

## 21. Use Appropriate Record Types

Use:

* `A` for IPv4
* `AAAA` for IPv6
* `CNAME` for aliases
* `MX` for mail routing
* `TXT` for verification and policies
* `SRV` for supported service discovery
* `PTR` for reverse DNS
* `CAA` to restrict certificate authorities where applicable

Do not overload TXT records for data better served by an application API.

---

## 22. Test Negative Caching Behavior

When creating a previously nonexistent name, users may continue receiving NXDOMAIN until negative caches expire.

Plan new hostnames before launch.

```text
Create DNS record
      |
      v
Wait for visibility
      |
      v
Enable application traffic
```

---

## 23. Respect DNS TTLs in Service Clients

Some applications resolve a hostname once at startup and never refresh it.

This defeats dynamic DNS changes.

Clients should:

* Re-resolve according to TTL
* Avoid caching forever
* Refresh connection pools
* Handle changed addresses
* Retry alternate endpoints safely

Behavior varies across language runtimes and networking libraries, so verify it explicitly.

---

## 24. Avoid Very Low TTLs Without Need

Extremely low TTLs can:

* Increase query volume
* Increase authoritative load
* Increase resolver dependency
* Provide little benefit when clients ignore low values
* Reduce resilience during DNS-provider outages

Use low TTLs intentionally, not automatically.

---

## 25. Document Recovery Procedures

Prepare runbooks for:

* Incorrect record deployment
* DNS-provider outage
* Registrar compromise
* DNSSEC failure
* Region failover
* Accidental zone deletion
* Expired domain registration
* Expired secondary nameserver configuration

DNS incidents can affect every service simultaneously.

---

## Common Mistakes

## 1. Assuming DNS Changes Are Instant

Resolvers and clients cache responses.

A record change may take effect gradually according to the previous TTL and negative caching behavior.

---

## 2. Lowering TTL at the Moment of Migration

Previously cached records still use the old TTL.

Lower TTL well before the planned change.

---

## 3. Using DNS as the Only Load Balancer

DNS cannot make per-request routing decisions and cannot immediately revoke cached answers.

Use a load balancer behind DNS for precise control.

---

## 4. Blocking TCP Port 53

Large or truncated DNS responses may require TCP.

Allow both UDP and TCP where appropriate.

---

## 5. Creating Long CNAME Chains

Long chains increase latency, complexity, and failure risk.

Keep alias chains short.

---

## 6. Leaving Dangling Records

DNS entries pointing to deleted cloud resources may create subdomain takeover risk.

Remove obsolete records immediately.

---

## 7. Using High TTLs for Frequently Changing Endpoints

High TTLs delay failover and migration.

Choose TTLs according to expected change frequency.

---

## 8. Using Extremely Low TTLs Everywhere

Very low TTLs increase DNS traffic and dependency on resolver availability without guaranteeing instant changes.

---

## 9. Assuming Round-Robin DNS Distributes Traffic Evenly

Caching, client selection, connection reuse, and unequal server capacity can produce uneven traffic.

---

## 10. Forgetting Negative Caching

A nonexistent hostname may remain cached as nonexistent even after the record is created.

Create records before launch and allow negative caches to expire.

---

## 11. Confusing DNSSEC With Encryption

DNSSEC validates DNS data authenticity.

It does not hide query names.

Use DoH or DoT for encrypted client-to-resolver communication.

---

## 12. Exposing Internal Names Publicly

Public DNS should not reveal private infrastructure unnecessarily.

Use private zones or split-horizon DNS.

---

## 13. Hardcoding IP Addresses

Hardcoded IP addresses break migrations, failover, and provider changes.

Use DNS names as stable identifiers.

---

## 14. Ignoring Client DNS Caching Behavior

Some runtimes cache DNS longer than the record TTL or resolve only once.

Test the actual client and connection-pool behavior.

---

## 15. Deleting Old Infrastructure Too Early

Clients may continue using cached old addresses.

Wait for the old TTL to expire and monitor residual traffic before decommissioning.

---

## Interview Questions

### 1. What happens when a client resolves a domain name?

The client checks local caches, then asks a recursive resolver. On a cache miss, the resolver queries root, TLD, and authoritative servers before returning and caching the final answer.

---

### 2. What is the difference between recursive and authoritative DNS?

A recursive resolver finds and caches answers for clients, while an authoritative server stores and returns the official DNS records for a zone.

---

### 3. What is TTL in DNS?

TTL defines how long a DNS response may be cached before it should be refreshed from an upstream server.

---

### 4. Why does DNS use both UDP and TCP?

UDP handles most small, low-overhead queries, while TCP supports large responses, truncated-response retries, zone transfers, and other cases requiring reliable transport.

---

### 5. Why is DNS failover not immediate?

Resolvers and clients may continue using cached records until their TTLs expire, and existing network connections may remain attached to the old endpoint.

---

## Key Takeaways

1. **DNS is a globally distributed naming and discovery system that maps stable domain names to changing infrastructure such as servers, load balancers, services, and CDN endpoints.**

2. **Reliable DNS design requires deliberate TTLs, redundant authoritative servers, secure account controls, monitoring, support for UDP and TCP, and careful handling of caching and migration behavior.**

3. **DNS is a critical routing layer but not a precise request-level load balancer; production systems should combine it with health-aware load balancers, resilient clients, and overlapping infrastructure during changes.**
