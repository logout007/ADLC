# Domain Specification 02: Clinical Encounters & Problem List

## 1. Domain Scope & Clinical Context
Governs outpatient, inpatient, and telehealth clinical encounters, chief complaints, structured SOAP progress notes (Subjective, Objective, Assessment, Plan), US Core Clinical Notes document references, and the longitudinal problem list.

---

## 2. Terminology & Coding Standards
- **Encounter Classification:** HL7 FHIR `Encounter` resource (act codes: `AMB` [ambulatory/outpatient], `VR` [virtual telehealth], `EMER` [emergency]).
- **US Core Required Clinical Notes (LOINC):**
  - **Progress Note:** LOINC `11506-3`
  - **History & Physical (H&P) Note:** LOINC `34117-2`
  - **Consultation Note:** LOINC `11488-4`
  - **Discharge Summary:** LOINC `18842-5`
  - **Emergency Department Note:** LOINC `34111-5`
- **Diagnosis & Billing Coding:** ICD-10-CM (e.g. `J45.909` - Unspecified asthma, uncomplicated).
- **Clinical Semantics:** SNOMED CT for longitudinal problem list concepts (e.g. `195967001` - Asthma).

---

## 3. Data Schema & PHI Sensitivity Classification

| Field Name | Type | Sensitivity Level | Description & Validation Rules |
| :--- | :--- | :--- | :--- |
| `encounterId` | UUID | Synthetic Identifier | Unique clinical encounter ID |
| `patientId` | UUID | Foreign Key | Target patient reference |
| `attendingProviderId` | UUID | Foreign Key | Clinician NPI reference |
| `encounterType` | Enum | Clinical Workflow | `OUTPATIENT` \| `TELEHEALTH` \| `INPATIENT` \| `EMERGENCY` |
| `status` | Enum | Lifecycle Status | `SCHEDULED` \| `IN_PROGRESS` \| `COMPLETED` \| `CANCELLED` |
| `chiefComplaint` | String | Sensitive PHI | Primary patient reason for visit in patient's words |
| `soapSubjective` | String | Highly Sensitive PHI | History of present illness (HPI), symptoms narrative |
| `soapObjective` | Object | Sensitive PHI | Physical exam findings & vital signs summary |
| `soapAssessment` | Array<ProblemDto> | Highly Sensitive PHI | Validated ICD-10 & SNOMED CT diagnosis list |
| `soapPlan` | String | Highly Sensitive PHI | Orders, prescriptions, referrals, follow-up instructions |
| `signedBy` | String | Regulatory Attestation | Licensed clinician NPI |
| `signedAt` | ISO DateTime | Compliance Timestamp | RFC 5905 NTP synchronized UTC signature timestamp |

---

## 4. Problem List Lifecycle Rules
- **Problem Statuses:** `ACTIVE` \| `INACTIVE` \| `RESOLVED` \| `REMISSION`.
- **Chronicity:** `CHRONIC` (ongoing monitoring) vs `ACUTE` (temporary episode).
- **Immutability After Signing:** Once an encounter is marked `COMPLETED` and signed by the clinician, the SOAP note text is locked. Corrections require an append-only addendum with separate timestamp and author NPI.

---

## 5. Personas & Scenarios (Given / When / Then)

### Scenario 1: Clinician Completes and Signs SOAP Note (Happy Path)
- **Given** clinician `dr_elena_vance` has an in-progress telehealth encounter with `p_sarah_connor`
- **When** the clinician submits `POST /api/encounters/:id/sign` with assessment ICD-10 `J45.909` (Asthma) and valid SOAP text
- **Then** the encounter transitions to `COMPLETED`
- **And** the condition "Asthma" is automatically updated on Sarah's active problem list
- **And** emits an ASTM E2147 audit event `ENCOUNTER_SIGNED` with clinician NPI and encounter ID.

### Scenario 2: Addendum to Completed Encounter After 48 Hours (Edge Case)
- **Given** an encounter completed and signed 48 hours ago
- **When** clinician `dr_elena_vance` submits `POST /api/encounters/:id/addendum` with supplemental pathology findings
- **Then** the original note remains locked and unchanged
- **And** an append-only `Addendum` record is persisted referencing the parent encounter with UTC timestamp
- **And** emits audit event `ENCOUNTER_ADDENDUM_ATTACHED`.

### Scenario 3: Missing Required Primary Diagnosis Blocks Signing (Failure Path)
- **Given** an in-progress encounter with subjective notes and exam findings
- **When** the clinician attempts to sign the encounter with zero diagnosis codes in `soapAssessment`
- **Then** the API rejects the request with HTTP 422 Unprocessable Entity and error code `PRIMARY_DIAGNOSIS_REQUIRED`
- **And** the encounter remains in `IN_PROGRESS` status until a valid ICD-10 code is attached.

### Scenario 4: Non-Attending Staff Attempts Direct Note Alteration (Misuse Path)
- **Given** an encounter marked `COMPLETED` and signed by `dr_elena_vance`
- **When** front-desk staff `fd_mark_miller` attempts `PUT /api/encounters/:id` to modify the plan
- **Then** the API returns HTTP 403 Forbidden with code `ENCOUNTER_LOCKED_SIGNED`
- **And** an alert `UNAUTHORIZED_CLINICAL_NOTE_MUTATION_ATTEMPT` is recorded in security telemetry.
