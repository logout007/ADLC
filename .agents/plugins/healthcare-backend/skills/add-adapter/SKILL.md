---
name: add-adapter
description: Standardized procedure to implement an external healthcare integration (EHR, FHIR, Lab, Billing) using an Anti-Corruption Layer (ACL) and contract fixtures.
---

# Skill: Add Healthcare Adapter

Follow this procedure whenever integrating an external service or API in healthcare (e.g., Epic/Cerner FHIR APIs, Quest/Labcorp results, Stripe/Healthcare billing).

## Principles
1. **Zero External PHI Leakage:** External services only receive data explicitly covered by a Business Associate Agreement (BAA) and defined in the spec.
2. **Contract Fixtures Over Live Sandboxes:** AI agents must NEVER call external live endpoints during development; integrate strictly against recorded synthetic contract fixtures.
3. **Anti-Corruption Layer (ACL):** External schemas must never bleed into core domain models. Always map external payloads through explicit adapters.
4. **Idempotency & Sequence Invariance:** All external webhooks, HL7 feeds, and dispatch events must be idempotent. Re-delivering an identical event MUST NOT duplicate clinical records or appointments.

## Implementation Steps

### 1. Spec Verification
- Confirm the external integration is documented in `docs/spec/`.
- Ensure the destination is on the approved vendor list (Layer 8). If unlisted, STOP and flag as a **Human-Only** area.

### 2. Define Contract Fixtures
- Store synthetic request and response fixtures in `fixtures/contracts/<service-name>/`.
- Validate that all fixture data uses synthetic identifiers (`p_synthetic_*`).
- Fixtures MUST include:
  - Standard request/response twin.
  - Duplicate replay fixture (asserting idempotency via unique external message/event ID).
  - Out-of-order or delayed event sequence fixture.

### 3. Implement Anti-Corruption Adapter
- Create `src/adapters/<service-name>.adapter.ts`.
- Implement two-way mapping:
  - `toExternalDto(domainEntity)`: Excludes all internal PHI fields not required by the external contract.
  - `toDomainEntity(externalDto)`: Normalizes external data into internal domain types.

### 4. Resiliency & Error Masking
- Mask raw upstream error payloads to avoid leaking external infrastructure details or patient names in logs.
- Wrap external calls in timeout, retry, and circuit-breaker handlers.

### 5. Audit Logging
- Emit an audit event for every external data transmission:
  - `action`: `EXTERNAL_DATA_DISPATCH`
  - `resourceId`: Internal resource ID
  - `actorId`: Triggering persona or system job
  - `destination`: External service name
  - `outcome`: `SUCCESS` or `FAILED`

### 6. Stop Conditions
- **HOLD:** If the external API requires new fields not documented in `docs/spec/`.
- **HOLD:** If third-party behavior differs from recorded contract fixtures.
