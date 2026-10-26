# Domain Specification 01: Identity, Demographics & HIPAA Consent

## 1. Domain Scope & Clinical Context
Governs patient identity resolution, Master Patient Index (EMPI) mapping, demographic capture, and mandatory HIPAA Notice of Privacy Practices (NPP) consent agreements. Applies to all patient intake, scheduling, and portal onboarding workflows.

---

## 2. Terminology & Coding Standards
- **Identifiers:** Internal UUID, Medical Record Number (MRN format: `MRN-[A-Z]{3}-[0-9]{6}`), Enterprise Master Patient Index (`EMPI-US-[0-9]{8}`).
- **Interoperability Standard:** HL7 FHIR R4 `Patient` resource complying with the official **US Core Patient Profile (v3.1.1/v4.0.0)**.
- **Consent Standards:** FHIR `Consent` resource (category: `http://terminology.hl7.org/CodeSystem/consentcategorycodes|hipaa-npp`, status: `active`).
- **Audit Standard:** ASTM E2147-18 & ONC § 170.315(d)(2) compliant immutable audit event.

---

## 3. Data Schema & PHI Sensitivity Classification

| Field Name | Type | Sensitivity Level | Description & Validation Rules |
| :--- | :--- | :--- | :--- |
| `patientId` | UUID | Synthetic Identifier | Internal unique identifier |
| `mrn` | String | Sensitive Identifier | Facility-unique Medical Record Number |
| `empiId` | String | Sensitive Identifier | Cross-facility enterprise master patient index |
| `legalName` | Object | PHI - Demographic | `{ prefix?: string, given: string[], family: string, suffix?: string }` |
| `preferredName` | String | Demographic | Patient preferred name / pronouns |
| `dateOfBirth` | ISO Date | PHI - Demographic | `YYYY-MM-DD` (Age verified against pediatric threshold $\le 18$) |
| `genderIdentity` | Enum | Demographic | `M` \| `F` \| `NON_BINARY` \| `OTHER` \| `UNDISCLOSED` |
| `contactDetails` | Object | Sensitive PHI | `{ phone: string, email: string, address: AddressDto }` |
| `nppConsentSigned` | Boolean | Compliance Flag | Must be `true` before non-emergency clinical scheduling |
| `nppConsentDate` | ISO DateTime | Compliance Timestamp | RFC 5905 NTP synchronized UTC timestamp of consent signature |
| `emergencyContact` | Object | PHI - Demographic | `{ name: string, relationship: string, phone: string }` |

---

## 4. Business Invariants & Compliance Rules
- **INV-ID-001 (Consent Before Service):** No elective encounter, prescription, or referral can be scheduled without active `nppConsentSigned = true`.
- **INV-ID-002 (EMPI De-duplication):** Incoming patient records matching on `(FamilyName + DOB + Phone)` or `(SSN_Hash)` must trigger identity reconciliation review before creating a new MRN.
- **INV-ID-003 (De-identification for Non-Clinical Roles):** Non-clinical staff (billing, IT, analytics) querying demographic statistics must receive de-identified quasi-identifiers (Safe Harbor: 3-digit Zip code, age year rather than exact DOB).

---

## 5. Personas & Scenarios (Given / When / Then)

### Scenario 1: Intake Onboarding with Signed Consent (Happy Path)
- **Given** synthetic patient `p_sarah_connor` accesses the digital intake portal
- **When** the patient submits `POST /api/intake/demographics` with valid name, DOB, and signs the HIPAA Notice of Privacy Practices (`nppConsentSigned: true`)
- **Then** the system returns HTTP 201 Created with a generated MRN (`MRN-SYN-884920`) and consent record with status `ACTIVE`
- **And** emits an ASTM E2147 audit event `PATIENT_IDENTITY_CREATED` with actor `p_sarah_connor`, resource `patientId`, and UTC timestamp.

### Scenario 2: Minor / Pediatric Intake Requiring Guardian Consent (Edge Case)
- **Given** synthetic patient `p_minor_timmy` with DOB indicating age 14
- **When** intake is submitted with `nppConsentSigned: true` but `guardianSignature` is null
- **Then** the system returns HTTP 422 Unprocessable Entity with error code `GUARDIAN_CONSENT_REQUIRED`
- **And** no active clinical profile is activated until verified legal guardian signature is provided.

### Scenario 3: Identity Collision & EMPI De-duplication Conflict (Failure Path)
- **Given** an existing confirmed patient record exists for `p_sarah_connor` with phone `(555) 019-2831`
- **When** an intake submission arrives with the exact same name, DOB, and phone, but differing address
- **Then** the automated creation halts with HTTP 409 Conflict and error code `EMPI_COLLISION_DETECTED`
- **And** the record is routed to the Medical Records Specialist queue without creating duplicate MRNs.

### Scenario 4: Non-Clinical Role Requests Demographic Export with Full SSN/DOB (Misuse Path)
- **Given** front-desk staff `fd_mark_miller` executes `GET /api/patients/:id/export`
- **When** the request attempts to query unmasked SSN, full birth date, and financial tokens
- **Then** the API intercepts the request, blocks unauthorized fields, and returns HTTP 200 with Safe-Harbor redacted fields (masked DOB year only, zero SSN)
- **And** logs a security warning event: `UNAUTHORIZED_PHI_FIELD_ACCESS_PREVENTED`.
