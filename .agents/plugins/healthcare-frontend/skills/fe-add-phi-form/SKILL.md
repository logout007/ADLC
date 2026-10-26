---
name: fe-add-phi-form
description: Standardized procedure to build HIPAA-compliant patient intake and clinical documentation forms with field-level sensitivity, validation, and zero client-side leakage.
---

# Skill: Frontend - Add PHI Form

Follow this procedure whenever implementing patient intake forms, medical history surveys, or clinical SOAP note documentation editors.

## HIPAA & Clinical Form Safeguards
1. **No Unencrypted Client Caching:** Never persist sensitive PHI (symptoms, SSN, diagnoses) in browser `localStorage` or unencrypted cookies. Use short-lived memory state or encrypted session storage.
2. **Explicit Consent Checkpoints:** Mandatory non-pre-checked checkboxes for HIPAA Notice of Privacy Practices consent.
3. **WCAG Accessible Form Controls:** Proper `<label for="...">`, ARIA invalid announcements, and clear error messaging.
4. **Zero PHI in URL Parameters:** Multi-step wizard forms must NEVER serialize patient names, SSNs, or inputs into URL query strings.
5. **Client Telemetry Masking:** Error reporters (Sentry/LogRocket) must scrub form values and input fields before dispatch.

## Implementation Steps

### 1. Schema Validation Definition
- Use schema validator (e.g., Zod) with strict sanitization:
  - Phone numbers formatted as `(XXX) XXX-XXXX`.
  - Date of Birth verified against minimum age requirements.
  - Required consent agreements validated as boolean `true`.

### 2. Implement Progressive Intake Steps
- Split complex clinical forms into bite-sized steps:
  - Step 1: Demographics & Emergency Contacts.
  - Step 2: Allergies & Medications.
  - Step 3: Medical History & Chief Complaint.
  - Step 4: HIPAA Consent Signature & Final Review.

### 3. Clear Feedback on Critical Allergies
- In allergy input fields, provide autocomplete against RxNorm/SNOMED allergen terms.
- Highlight severe reactions (e.g. Anaphylaxis) with clear visual and screen-reader badges.

### 4. Direct Presigned Object Storage (S3) Upload Workflow
- For medical files, documents, and lab photos:
  - Step 1: Request presigned upload URL from backend (`POST /patient/documents/upload-url` with file content type).
  - Step 2: Upload file directly to object store via `PUT` request with `Content-Type` header (bypassing backend server).
  - Step 3: Call backend confirmation endpoint (`POST /patient/documents/confirm-upload`) with `s3Key`.
- **Zero Local Storage:** Never cache binary image/PDF files in browser storage.

### 5. Document Extraction & Medication Reconciliation
- For prescription/report extractions:
  - Present extracted items for patient review.
  - Check against existing active medications; flag duplicates and display `alreadyCommitted` lines.
  - Require explicit patient confirmation before merging extracted lines into the active medication regimen.

### 6. Memory Scrubbing on Submit / Reset
- On form submission or modal close, purge sensitive in-memory form values.
- Prevent browser autocomplete on sensitive fields (`autocomplete="off"` for SSN, Aadhaar, and clinical notes).

### 7. Stop Conditions
- **HOLD:** If the spec allows submitting intake without explicit consent confirmation.
- **HOLD:** If file uploads stream raw PHI document binaries directly through standard application APIs instead of direct presigned object storage.
