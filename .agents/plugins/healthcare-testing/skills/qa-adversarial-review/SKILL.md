---
name: qa-adversarial-review
description: Standardized procedure to mechanically audit Pull Requests and code changes for bypassed data projections, missing audit logs, and race conditions before merge.
---

# Skill: Adversarial QA & Safety Audit Review

Follow this procedure to mechanically audit any pull request or code change before approval. This review adopts an **adversarial stance** (assuming the code contains hidden privacy leaks or missing compliance hooks).

## Pre-conditions
1. All deterministic linters (`npm run lint`, `npm run typecheck`) have passed cleanly.
2. The code touches API endpoints, services, or data models.

## The 3 Adversarial Audit Rules

### Rule 1: Bypassed Data Projections (Data Leakage Hunt)
- **What to look for:** Does any route return raw domain objects without passing through explicit role whitelisting?
- **Failure Condition:** If `clinicalNotes` or sensitive fields can reach a `front_desk` or non-clinical actor, the review **MUST FAIL** immediately.
- **Remedy:** Wrap all responses in strict DTO whitelists (`projectAppointmentByRole`).

### Rule 2: Missing Audit Logs (HIPAA Compliance Hunt)
- **What to look for:** Does any read or mutation of patient data happen without emitting an audit event?
- **Failure Condition:** If an endpoint queries or modifies patient records without calling `recordAuditEvent()`, the review **MUST FAIL** immediately.
- **Remedy:** Emit an immutable audit event with `{ actorId, actorRole, action, resourceId, outcome }` linked into the SHA-256 hash chain.

### Rule 3: Asynchronous Race Conditions (Concurrency Hunt)
- **What to look for:** Are availability checks and booking/mutation steps decoupled without collision detection?
- **Failure Condition:** If two simultaneous requests can claim the same slot, the review **MUST FAIL** immediately.
- **Remedy:** Enforce atomic slot locking or deterministic conflict status checks (`409 Conflict`).

## Execution Command
```bash
npm run qa:adversarial-review
```
