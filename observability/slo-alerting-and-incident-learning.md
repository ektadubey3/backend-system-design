# SLO Alerting and Incident Learning

## TL;DR

Page on actionable user impact or imminent exhaustion of a safety margin, diagnose with dependency/saturation signals, and correlate changes. Multi-window SLO burn alerts balance speed with noise. Incidents should produce tested changes to limits, automation, telemetry, or architecture—not blame-oriented narratives.

## Alert hierarchy

1. **Page:** urgent human action can reduce material user harm now.
2. **Ticket:** a safety margin or slow burn needs planned work.
3. **Dashboard/exploration:** supports understanding but requires no notification.
4. **Automated control:** an expected condition with a safe tested response.

Every alert has owner, severity, user impact, query/dashboard, immediate actions, escalation, and a condition for resolution. If no useful action exists, improve the automation or remove the page.

## Symptom and cause

SLO burn, failed critical journeys, or expired async work are symptoms. CPU, pool saturation, replica lag, partition skew, and dependency errors are causes or risk indicators. Page on symptoms when possible and display causes in the incident view. Also alert directly on a hard safety margin—such as disk/log retention about to make recovery impossible—before user failure.

Multi-window burn rules use a short window for responsiveness and a longer one to confirm sustained consumption. Pair fast/high-burn pages with slower/lower-burn tickets. Test alert math against no-traffic and low-volume periods.

## Incident dashboard

Organize by:

- user impact and affected cohorts;
- request/event rate, error, latency, completion/age;
- saturation, admission, shedding, retries, and fallback;
- dependencies, regions/zones, versions, and partitions;
- recent deploy/config/feature/traffic changes;
- telemetry pipeline gaps.

Do not require responders to join twenty unrelated dashboards. Provide drill-down links and known-good comparisons.

## Runbooks and automation

Runbooks state how to validate the alert, bound blast radius, apply reversible mitigations, protect data, escalate, and verify recovery. Commands that mutate production should be safe, scoped, audited, and preferably automated with guardrails. A rollback is not safe if schema/data compatibility was not designed.

## Post-incident learning

Build a factual timeline from user signals, decisions, and system changes. Explain contributing conditions and why defenses did not prevent or limit impact. Actions should have owners and verification: a load test, restore drill, failure injection, alert test, safer rollout, capacity limit, or architecture change.

Track recurring patterns across incidents. A “human error” conclusion usually hides missing guardrails, ambiguous interfaces, excessive privilege, or an unrehearsed procedure.

## Failure modes

- One page per host creates a storm during fleet failure.
- SLO alert excludes retries or reports queue acceptance as completion.
- Alert auto-closes when traffic drops to zero.
- Dashboard averages hide one failed region/tenant tier.
- Runbook depends on the unavailable identity or control plane.
- Postmortem action says “be more careful” with no testable control.

## Interview prompts

- What wakes a human versus creates a ticket?
- Which safety margin should page before users fail?
- How will responders know a deploy changed the system?
- How is an incident action verified and kept from regressing?

## Two-minute answer

Define user-centered SLIs and use multi-window burn-rate alerts: fast severe burn pages, slow risk creates tickets. Pair symptom alerts with a compact diagnostic view of rate/errors/latency, saturation, retries/shedding, dependencies, cohorts, and recent changes. Each page has an owned runbook with reversible containment and recovery validation. Preserve audit/change events, make telemetry failure visible, and turn incidents into owned, testable controls such as failure injection, capacity limits, restore drills, or safer rollout.

## References

- [Google SRE Workbook — Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)

