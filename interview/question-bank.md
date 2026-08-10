# System Design Question Bank

## Foundation Designs

1. URL Shortener
2. Distributed Rate Limiter
3. Key-Value Store
4. Distributed Cache
5. Pastebin
6. Unique ID Generator

## Event-Driven Systems

7. Notification Service
8. Email Delivery Platform
9. Webhook Delivery Service
10. Analytics Event Pipeline
11. Job Scheduler

## Realtime Systems

12. Realtime Chat
13. Presence Service
14. Collaborative Document Editing
15. Live Sports Score Updates

## Feed and Search

16. News Feed
17. Search Autocomplete
18. Full-Text Search Service
19. Trending Topics

## Commerce

20. Shopping Cart
21. Payment System
22. Ticket Booking
23. Inventory Reservation
24. Food Delivery
25. Ride Matching

## Storage and Media

26. Dropbox / Drive
27. Photo Storage
28. Video Streaming
29. Live Streaming

## Senior-Level Follow-ups

For any design, add one constraint:

- 10× traffic tomorrow;
- one region fails;
- one tenant generates 40% of traffic;
- duplicate events are common;
- strict data residency is required;
- writes must survive a region loss;
- p99 latency doubles while p50 is stable;
- the queue is six hours behind;
- a cache cluster is unavailable;
- database replication lag reaches 30 seconds.
