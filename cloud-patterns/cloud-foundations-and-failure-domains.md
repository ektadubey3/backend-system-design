# Cloud Foundations and Failure Domains

## TL;DR

Design across explicit failure domains: process, host, zone, region, account/project, and provider control plane. Separate data-plane serving from control-plane changes, understand managed-service durability and quotas, and test behavior when APIs, identity, or a zone are unavailable. Multi-zone labels do not prove application-level resilience.

## Regions, zones, and placement

A region is a geographic deployment area; zones are intended as separate infrastructure failure domains within it. Provider definitions and service behavior vary, so verify guarantees. Spread redundant instances and data copies across zones and keep enough capacity to carry the declared loss.

One load balancer across two zones is not sufficient if all database replicas, NAT paths, secrets, or message partitions share one failure domain. Map the entire critical dependency graph.

## Control plane and data plane

The data plane handles current requests using existing configuration. The control plane creates resources, changes routes, schedules workloads, rotates configuration, or assigns ownership. A resilient service should continue serving an established configuration through a bounded control-plane outage where possible.

Avoid per-request calls to provisioning, secrets-management, deployment, or global routing APIs. Cache/lease safe configuration with versions and define its expiry/fail behavior.

## Managed services and shared responsibility

A managed database or queue can provide replacement, patching, replication, and backups under its contract. The application still owns schema, access policy, client timeouts/retries, partition keys, capacity/quota, retention, restore tests, and correctness during failover.

Ask for each service:

- acknowledgement and consistency semantics;
- zonal/regional placement and correlated dependencies;
- failover triggers and expected interruption/data-loss window;
- scaling ceilings, quotas, throttling, and hot-key behavior;
- backup/export/restore and exit path;
- maintenance and version policy;
- observability and audit availability.

## Identity, accounts, and blast radius

Use separate accounts/projects/subscriptions and roles to isolate environments and critical cells. Central identity simplifies governance but can become a global recovery dependency. Keep break-glass access protected, audited, and tested. Apply policy as code with review and prevent workloads from obtaining broad metadata/management credentials.

## Cost as an architectural constraint

Model steady and peak compute, storage copies, IOPS, requests, egress across zones/regions/internet, logging, backups, and idle failover headroom. Autoscaling does not cap spend; quotas and budgets do. Egress and request charges can change the optimal placement and cache strategy.

FinOps metrics should map cost to product/tenant/workload without introducing unbounded observability labels.

## Failure modes

- Instances span zones but a zonal database or NAT path does not.
- Control-plane outage prevents autoscaling exactly when capacity is needed.
- Provider quota is discovered only during regional failover.
- Managed backup exists but keys/permissions/restore time are untested.
- All environments share one account-level credential or quota.
- Cost optimization removes the headroom required for one-zone loss.

## Interview prompts

- Which failures are independent, and which provider services are regional/global?
- Can serving continue if deployment/identity/config control planes fail?
- Does surviving capacity meet the peak after a zone loss?
- Which quota or egress path fails during evacuation?

## Two-minute answer

Map every critical component across process, host, zone, region, account, and control-plane boundaries. Place redundant capacity and data across independent zones and size survivors for the declared failure. Keep per-request data-plane work independent of management APIs and cache versioned configuration safely. For managed services, verify durability, consistency, failover, quota, backup/restore, and exit semantics. Isolate environments/cells with scoped identity, model egress and telemetry cost plus failover headroom, and test zone/control-plane/restore scenarios.

