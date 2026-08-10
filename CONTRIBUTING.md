# Contributing

Contributions should improve correctness, interview usefulness, or production reasoning. More content is not automatically better content.

## Canonical Chapter Structure

Use this structure for major technical chapters:

```text
# Topic

## Interview TL;DR
## Mental Model
## How It Works
## Architecture
## When To Use It
## When Not To Use It
## Scaling Characteristics
## Failure Modes
## Trade-offs
## Production Gotchas
## Alternatives
## Interview Questions
## 2-Minute Interview Answer
## Senior-Level Follow-ups
## References
```

Not every small note needs every heading, but deep-dive chapters should cover the same concerns.

## Technical Standard

Avoid categorical shortcuts such as:

- SQL = consistency, NoSQL = scalability
- microservices = scalable
- horizontal scaling = unlimited
- Redis = always a cache
- Kafka = exactly-once by default
- retries = reliability

Prefer conditional reasoning:

> Given requirement X and constraint Y, option A gives us benefit B at the cost of C. We mitigate C with D. We would reconsider A if metric or requirement E changes.

## Case Study Standard

Every complete design should include:

1. Prompt
2. Clarifying questions
3. Functional requirements
4. Non-functional requirements
5. Back-of-the-envelope estimates
6. API design
7. Data model
8. High-level architecture
9. Critical request/data flows
10. Deep dives
11. Scaling plan
12. Failure scenarios
13. Consistency semantics
14. Security
15. Observability
16. Trade-offs
17. Evolution triggers
18. Interview follow-ups

## References

Prefer authoritative sources:

- Standards and RFCs
- Official database/broker documentation
- Research papers
- Cloud architecture documentation
- High-quality engineering blogs for implementation experience

Do not add references merely to make an article look authoritative.

## Pull Requests

Keep changes focused. Explain:

- what changed;
- why it improves the repository;
- which interview or production misconception it addresses;
- how the documentation was validated.
