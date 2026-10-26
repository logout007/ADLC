---
name: be-add-scheduling
description: Standardized procedure to build appointment scheduling, provider availability checking, conflict detection, and visit lifecycle tracking.
---

# Skill: Backend - Scheduling & Conflict Detection

Follow this procedure whenever implementing appointment booking, provider schedules, or calendar management.

## Principles
1. **Consent-Gated Booking:** Patient treatment consent must be active prior to booking non-emergency visits.
2. **Deterministic Conflict Prevention:** Atomic database operations / transactional locks prevent double-booking.
3. **Strict Minimum Necessary:** Scheduling queries from non-clinical staff must never leak clinical notes or reasons for visit.

## Implementation Steps

### 1. Spec & Calendar Rules Check
- Confirm slot duration, operating hours, and buffer times in `docs/spec/appointment_booking.md`.

### 2. Implement Availability Search
- Query provider calendar for blocked slots, existing confirmed appointments, and holidays.
- Return available slot windows without exposing other patients' booking metadata.

### 3. Implement Booking with Transactional Lock
- Verify patient consent status (`nppConsentSigned = true`).
- Execute atomic check-and-insert transaction:
  - If slot already booked with status `CONFIRMED`: Return HTTP 409 `SLOT_UNAVAILABLE` (masking other patient PHI).
  - Else: Insert new appointment with status `SCHEDULED`.

### 4. Lifecycle Management
- Support transition endpoints: `CHECK_IN`, `COMPLETED`, `NO_SHOW`, `CANCELLED`.
- Require cancellation reason.

### 5. Audit Logging
- Emit audit event: `action: APPOINTMENT_SCHEDULED`, `patientId`, `providerId`, `slot`.

### 6. Stop Conditions
- **HOLD:** If the spec does not specify the cancellation window or late-cancellation policies.
