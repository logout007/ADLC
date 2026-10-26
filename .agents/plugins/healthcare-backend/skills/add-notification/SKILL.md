---
name: add-notification
description: Standardized procedure to implement HIPAA-compliant patient and provider notifications (SMS, Email, Push) with zero PHI leakage.
---

# Skill: Add Healthcare Notification

Follow this procedure whenever implementing automated or triggered notifications to patients or care team members.

## HIPAA Compliance Rule
Under HIPAA and TCPA, unencrypted notification channels (SMS text messages, standard email, push notifications) must **NEVER** contain Protected Health Information (PHI).
- **Prohibited in SMS/Email:** Diagnoses, medication names, clinician specialty (if revealing condition like oncology/psychiatry), lab results, test names.
- **Allowed:** Generic notices ("You have a new message in your patient portal", "Your upcoming appointment has been scheduled") + deep link to authenticated portal.

## Implementation Steps

### 1. Spec & Consent Check
- Verify that patient communication preferences (SMS/Email opt-in) are verified before dispatch.
- Confirm template text matches an approved template in `docs/spec/`.

### 2. Template Payload Sanitation
- Ensure message builder strictly whitelists template tokens.
- Block dynamic inclusion of free-text clinical notes or diagnosis codes.
- Only include:
  - Patient first name (synthetic in dev: `Sarah`)
  - Date and time of appointment
  - Generic portal link (`https://portal.clinic.test/messages/msg_123`)

### 3. Mask Destination Addresses in Telemetry
- Redact synthetic phone numbers (`(555) ***-1234`) and emails (`s***@synthetic.test`) before logging dispatch events.

### 4. Emit Audit Event
- Emit an immutable audit record:
  - `action`: `NOTIFICATION_DISPATCHED`
  - `actorId`: Triggering persona / system worker
  - `channel`: `SMS` | `EMAIL` | `PUSH`
  - `recipientId`: `p_sarah_connor`
  - `templateId`: `APPT_CONFIRMATION_V1`
  - `outcome`: `SUCCESS` | `FAILED`

### 5. Stop Conditions
- **HOLD:** If a feature request asks to put clinical details, test results, or doctor specialty into an SMS/Email body without authenticated portal login.
