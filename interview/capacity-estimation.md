# Capacity Estimation

Estimate only quantities that can change the architecture.

## Requests Per Second

```text
average RPS = daily requests / 86,400
peak RPS = average RPS × peak factor
```

A peak factor of 5-10× can be a reasonable interview assumption if traffic shape is unknown; state that it is an assumption.

## Storage

```text
storage/day = writes/day × bytes/write
storage/year = storage/day × 365
```

Then account for:

- replication factor;
- indexes;
- metadata;
- retention;
- compression.

## Bandwidth

```text
egress/sec ≈ response RPS × average response bytes
```

This can justify a CDN, compression, media transcoding, or regional delivery.

## Concurrent Connections

For realtime systems:

```text
concurrent connections ≠ requests per second
```

Estimate simultaneously connected clients, heartbeat cost, fan-out, and reconnect bursts.

## Example — URL Shortener

Assume:

- 100 million new short URLs/month;
- 100:1 redirect-to-create ratio;
- 500 bytes stored per mapping including metadata/index overhead before replication.

Questions the estimate should answer:

- Is the dataset memory-resident?
- Is one database partition enough?
- Is caching worth it?
- Is key generation centralized or distributed?
- Is the redirect path more important than the creation path?

Do not spend interview time calculating numbers that will not affect a design choice.
