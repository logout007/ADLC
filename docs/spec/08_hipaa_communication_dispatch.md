# Specification: HIPAA-Compliant Communication & Notification Dispatch

## 1. Feature Intent & Goal
Provide automated SMS and email notifications to synthetic patients regarding appointments, intake status updates, and lab completion notices without transmitting Protected Health Information (PHI) over unencrypted channels.

---

## 2. Personas (Synthetic Only)
1. **Patient (`p_sarah_connor`)**:
   - Role: `patient`
   - Allowed: Receive notifications, manage opt-in/opt-out communication preferences.
2. **Notification Dispatch Worker (`system_worker_queue`)**:
   - Role: `system`
   - Allowed: Process outbound dispatch queue, call external messaging adapter with scrubbed templates.
3. **Clinical Coordinator (`coordinator_alex_chen`)**:
   - Role: `clinical_staff`
   - Allowed: Trigger automated portal message notification ("New message from Dr. Vance").

---

## 3. HIPAA Communication Rules & Allowed Content

| Communication Channel | PHI Permitted? | Allowed Content | Prohibited Content |
| :--- | :--- | :--- | :--- |
| **SMS (Text Message)** | **STRICTLY NO** | Patient first name, appointment date/time, clinic location, link to secure portal | Diagnosis, clinical specialty (e.g. Oncology, Psychiatry), test names, medications |
| **Email (Standard)** | **STRICTLY NO** | Generic alert ("You have a secure message waiting"), link to portal login | Clinical notes, lab values, physician notes, prescription details |
| **Authenticated Portal** | **YES** (Covered by Auth & BAA) | Full clinical notes, lab results, diagnoses | N/A (Protected behind MFA & Auth) |

---

## 4. Scenarios (Given / When / Then)

### Scenario 1: Appointment Scheduled Notification (Happy Path)
- **Given** synthetic patient `p_sarah_connor` has an active SMS opt-in preference
- **When** an appointment is confirmed for `2026-10-15T10:00:00Z`
- **Then** an SMS payload is generated using template `APPT_CONFIRMATION_V1`:
  > *"Hi Sarah, your appointment with Dr. Elena Vance is confirmed for Oct 15 at 10:00 AM. View details: https://portal.clinic.test/m/123"*
- **And** no clinical reason or notes are included in the SMS body
- **And** an ASTM E2147 audit event `NOTIFICATION_DISPATCHED` is recorded.

### Scenario 2: Patient Opted Out (Edge Case)
- **Given** synthetic patient `p_sarah_connor` has `smsOptIn: false`
- **When** a dispatch job attempts to send an SMS reminder
- **Then** the job halts dispatch with status `SUPPRESSED_OPT_OUT`
- **And** an audit log records `NOTIFICATION_SUPPRESSED`.

### Scenario 3: Messaging Gateway Network Outage (Failure Path)
- **Given** external SMS carrier / Twilio gateway returns HTTP 503 Service Unavailable
- **When** the notification worker attempts dispatch
- **Then** the worker catches the error without crashing, logs an incident, and moves the message to the dead-letter retry queue with exponential backoff (1m, 5m, 15m)
- **And** never exposes patient destination phone number in plain text in the error log.

### Scenario 4: Misuse Attempt - Clinical Details in Unencrypted Channel (Misuse Path)
- **Given** an automated workflow or third-party integration attempts to populate an SMS body with `clinicalNotes` or `diagnosis`
- **When** the payload validation pipeline evaluates the message
- **Then** the pipeline rejects the dispatch immediately with validation error `PHI_LEAK_PREVENTED`
- **And** an alert is emitted to the security audit logger and message is blocked.

---

## 5. Traceability & Implementing Skill
- Skill: `.agents/plugins/healthcare-backend/skills/add-notification/SKILL.md`
- Audit Policy: Immutable logging of recipient synthetic ID, template ID, and channel.
