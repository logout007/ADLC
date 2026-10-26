---
name: fe-add-audit-inspector
description: Standardized procedure to implement client-side ASTM E2147-18 tamper-evident chained audit logging and verification inspection modal for healthcare demonstrations and compliance testing.
---

# Skill: Frontend - ASTM E2147-18 Audit Inspector & Demo Chassis

Follow this procedure when implementing demonstrable compliance logging, tamper-evident audit trails, or mobile-first simulation wrappers.

## Compliance Safeguards
1. **ASTM E2147-18 Standard:** Audit entries must record `timestamp`, `actorId`, `actorRole`, `action`, `resourceType`, `resourceId`, and `details`.
2. **Tamper-Evident Hashing:** Each audit entry links cryptographically to the previous entry using a chained hash (`sha256-<hex>` or similar).
3. **Zero PHI in Audit Payloads:** Ensure audit entries describe actions and opaque resource IDs without logging sensitive clinical notes or unmasked identifiers.

## Implementation Steps

### 1. In-Memory Chained Audit Logger (`useTamperAudit`)
- Implement a lightweight state hook or context provider maintaining an immutable array of audit events.
- On each mutation or patient data access (profile switch, vital logged, document viewed, consent toggled), dispatch:
  ```javascript
  logAuditEvent(action, resourceType, resourceId, details)
  ```
- Compute the chained hash linking from the preceding event's hash.

### 2. Audit Trail Inspector Modal (`AuditModal`)
- Accessible via a prominent inspector button in development/demo mode.
- Render:
  - Total event count and cryptographic integrity status.
  - Event log table with timestamp, actor role, action badge, and short details.
  - Cryptographic hash badge showing chaining links.
  - Filter by action or resource type.

### 3. Viewport Chassis Preview Wrapper
- For mobile-first web applications running in desktop browsers:
  - Implement a preview chassis toggle allowing quick switching between **Mobile View** (fixed 440px phone container) and **Expanded Desktop View**.
  - Provide a top toolbar with demo controls (audit modal trigger, active profile indicator, and viewport toggle).

### 4. Stop Conditions
- **HOLD:** If the audit logging mechanism exposes unredacted patient clinical notes or raw identifiers.
