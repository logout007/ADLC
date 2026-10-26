# Specification: Patient Digital Intake & External EHR Sync

## 1. Feature Intent & Goal
Provide an authenticated digital intake workflow for synthetic patients before their first clinic visit. The system captures demographics, emergency contacts, medical history, and HIPAA consent, and syncs demographic records with an external hospital EHR via a FHIR R4 Patient adapter.

---

## 2. Personas (Synthetic Only)
1. **New Patient (`p_sarah_connor`)**:
   - Role: `patient`
   - Allowed: Submit intake form, sign HIPAA privacy acknowledgement, view own intake status.
2. **Medical Records Specialist (`records_clerk_davis`)**:
   - Role: `records_staff`
   - Allowed: View synchronization status, retry failed FHIR sync jobs.
   - Forbidden: View raw psychiatry or substance-use clinical intake sections.
3. **Primary Care Physician (`dr_elena_vance`)**:
   - Role: `clinician`
   - Allowed: Full access to complete intake history, medical history, and clinical questionnaire.

---

## 3. Data Schema & PHI Tagging

| Field | Type | Tag | Access Policy |
| :--- | :--- | :--- | :--- |
| `patientId` | String | Synthetic Identifier | Internal & External FHIR |
| `legalName` | Object | PHI - Demographic | Patient, Clinician, Records Staff |
| `dateOfBirth` | String (ISO) | PHI - Demographic | Patient, Clinician, Records Staff |
| `contactEmail` | String | Synthetic Demographic | Scrubbed in external logs |
| `consentSigned` | Boolean | Compliance Flag | All roles |
| `pastSurgeries` | Array | Sensitive PHI - Clinical | Clinician Only |
| `symptomsDescription`| String | Sensitive PHI - Clinical | Clinician Only |

---

## 4. Scenarios (Given / When / Then)

### Scenario 1: Successful Intake Submission & FHIR Sync (Happy Path)
- **Given** synthetic patient `p_sarah_connor` accesses the digital intake portal
- **When** the patient signs the HIPAA consent agreement and submits demographic details and past medical history via `POST /api/intake`
- **Then** the intake record is persisted with status `SUBMITTED`
- **And** the EHR adapter transforms the demographic payload into a US Core FHIR R4 `Patient` resource
- **And** an ASTM E2147 audit event `INTAKE_SUBMITTED_AND_SYNCED` is recorded.

### Scenario 2: Unsigned Consent Blocks Sync (Edge Case)
- **Given** an intake payload is submitted with `consentSigned: false`
- **When** the intake handler validates the submission
- **Then** the request is rejected with HTTP 422 Unprocessable Entity and error code `CONSENT_UNACKNOWLEDGED`
- **And** zero data is transmitted to the external EHR adapter.

### Scenario 3: External EHR Adapter Timeout / Circuit Breaker (Failure Path)
- **Given** external hospital EHR FHIR gateway is experiencing high latency or downtime
- **When** the intake submission attempts outbound FHIR sync
- **Then** the adapter hits a 3000ms timeout threshold, activates the circuit breaker, and transitions intake sync status to `PENDING_RETRY`
- **And** patient submission succeeds locally without data loss
- **And** queues an idempotent background sync job.

### Scenario 4: Records Staff Query Clinical Symptoms (Misuse Path)
- **Given** a submitted intake record exists containing both demographic and sensitive clinical symptoms
- **When** medical records staff `records_clerk_davis` queries the intake summary via `GET /api/intake/:id`
- **Then** the response contains demographic fields (`legalName`, `dob`, `syncStatus`)
- **And** clinical fields (`pastSurgeries`, `symptomsDescription`) are strictly stripped from the payload.

---

## 5. External Integration Boundaries
- External FHIR endpoint: Mocked via contract fixture in `fixtures/contracts/fhir/`.
- No live network requests to third-party endpoints during development or automated runs.
- Any change to the FHIR patient mapping schema requires engineering lead review.
