# Specification: Patient Appointment Booking & RBAC Access Control

## 1. Feature Intent & Goal
Enable synthetic patients to sign mandatory treatment consent and schedule medical appointments. Front-desk staff must be able to view appointment scheduling metadata to manage clinic flow, but under no circumstance view clinical notes or diagnoses. Clinicians have full access to clinical notes and history.

---

## 2. Personas (Synthetic Only)
1. **Patient (`p_sarah_connor`)**:
   - Role: `patient`
   - Allowed: Sign consent, book appointments, view own bookings.
   - Forbidden: View appointments of other patients, modify clinician notes.
2. **Front-Desk Staff (`fd_mark_miller`)**:
   - Role: `front_desk`
   - Allowed: View clinic schedule (appointment time, patient name, status, clinician name).
   - Forbidden: View clinical notes, symptoms, or diagnoses under any circumstance.
3. **Clinician (`dr_elena_vance`)**:
   - Role: `clinician`
   - Allowed: View full appointment details including clinical notes, medical reason, and diagnosis notes.

---

## 3. Scenarios (Given / When / Then)

### Scenario 1: Happy Path (Patient books after signing consent)
- **Given** synthetic patient `p_sarah_connor` has signed an active consent agreement
- **When** the patient submits a booking request for slot `2026-10-15T10:00:00Z` with clinician `dr_elena_vance` and notes "Routine annual respiratory checkup"
- **Then** the booking is created with status `CONFIRMED`
- **And** an audit event `APPOINTMENT_BOOKED` is recorded with actor `p_sarah_connor`.

### Scenario 2: Edge Case (Booking attempted with expired / missing consent)
- **Given** synthetic patient `p_sarah_connor` has NOT signed consent or consent is revoked
- **When** the patient attempts to book an appointment
- **Then** the system rejects the booking with HTTP 403 Forbidden and error code `CONSENT_REQUIRED`
- **And** no appointment record is persisted.

### Scenario 3: Failure Path (Double booking / Slot conflict)
- **Given** clinician `dr_elena_vance` already has a confirmed booking at `2026-10-15T10:00:00Z`
- **When** another patient attempts to book the same slot with `dr_elena_vance`
- **Then** the system rejects the request with HTTP 409 Conflict and error code `SLOT_UNAVAILABLE`
- **And** no PHI regarding the existing booking is leaked in the error response.

### Scenario 4: Misuse Path (Front-desk attempts to view clinical notes)
- **Given** a confirmed appointment exists with clinical notes "Patient reports recurring chest tightness and mild dyspnea"
- **When** front-desk staff `fd_mark_miller` requests the appointment details via `GET /api/appointments/:id`
- **Then** the system returns HTTP 200 with appointment metadata (patient name, slot, status, clinician)
- **And** the field `clinicalNotes` is strictly omitted from the response
- **And** an audit event `APPOINTMENT_VIEWED` is recorded indicating non-clinical role view.

---

## 4. Compliance Invariants
- **INV-001 (Consent Invariant):** An appointment must NEVER be persisted without valid, active consent timestamped prior to booking.
- **INV-002 (RBAC Redaction Invariant):** The response payload for role `front_desk` must NEVER contain the string keys or values of `clinicalNotes`.
- **INV-003 (Audit Trail Invariant):** Every read and mutation of patient records must produce an immutable audit log entry.
