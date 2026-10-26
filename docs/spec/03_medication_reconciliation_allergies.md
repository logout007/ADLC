# Domain Specification 03: Medication Reconciliation & Allergies

## 1. Domain Scope & Clinical Context
Governs active medications, electronic prescription order entry, Drug-Drug Interaction (DDI) checking, patient allergy cross-reactivity, medication reconciliation across transitions of care, and NCPDP SCRIPT e-prescribing standards.

---

## 2. Terminology & Coding Standards
- **Medication Terminology:** RxNorm Concept Unique Identifiers (RxCUI) for clinical drugs, National Drug Code (NDC) for manufactured packages.
- **Allergies & Adverse Reactions:** SNOMED CT allergen concepts, HL7 FHIR `AllergyIntolerance` resource.
- **E-Prescribing Standards:** **NCPDP SCRIPT Standard v2017071** (transactions: `NewRx`, `RxChangeRequest`, `RxChangeResponse`, `CancelRx`).
- **Drug Utilization Review (DUR):** Severity levels: `CONTRAINDICATED` (Level 1), `MAJOR` (Level 2), `MODERATE` (Level 3), `MINOR` (Level 4).

---

## 3. Data Schema & PHI Sensitivity Classification

| Field Name | Type | Sensitivity Level | Description & Validation Rules |
| :--- | :--- | :--- | :--- |
| `prescriptionId` | UUID | Synthetic Identifier | Unique prescription order ID |
| `patientId` | UUID | Foreign Key | Target patient reference |
| `prescriberNpi` | String | Regulatory ID | DEA / NPI of licensed prescriber |
| `rxCui` | String | Medication Code | RxNorm Concept Identifier (e.g. `866414` for Albuterol HFA) |
| `drugName` | String | Sensitive PHI | Display name with strength (e.g. "Albuterol 90mcg Inhalation Aerosol") |
| `dosageForm` | String | Clinical Detail | Tablet, Inhaler, Oral Solution, Injection |
| `sig` | String | Highly Sensitive PHI | Patient instructions (e.g. "Inhale 2 puffs every 4-6 hours PRN") |
| `quantity` | Number | Clinical Detail | Dispensed quantity (e.g. `1 inhaler` or `30 tablets`) |
| `refillsAllowed`| Integer| Regulatory Detail | Refill count (DEA Schedule II drugs MUST be 0) |
| `isControlled` | Boolean | Regulatory Flag | DEA Controlled Substance Schedule II-V flag |
| `prescribedAt` | ISO DateTime | Compliance Timestamp | RFC 5905 NTP synchronized UTC order timestamp |

---

## 4. Clinical Safety Invariants
- **INV-MED-001 (Drug-Allergy Fatal Check):** A prescription order MUST run real-time cross-reactivity against active patient allergies (e.g., Cephalosporin prescribed to a patient with severe Anaphylactic Penicillin allergy). If severity = `SEVERE` / `LIFE_THREATENING`, order entry is blocked unless explicit clinical justification is signed.
- **INV-MED-002 (Drug-Drug Major Interaction):** Automated check against all current active medications. Level 1 (`CONTRAINDICATED`) interactions trigger an interruptive clinical hard stop.
- **INV-MED-003 (Schedule II DEA Safeguards):** Controlled substance prescriptions require 2-factor clinician attestation and cannot have automated refills.

---

## 5. Personas & Scenarios (Given / When / Then)

### Scenario 1: New E-Prescription with Automated Safety Validation (Happy Path)
- **Given** clinician `dr_elena_vance` is preparing an electronic prescription for `p_sarah_connor`
- **When** the clinician submits `POST /api/prescriptions` with RxNorm `866414` (Albuterol) with zero conflicting allergies or interacting drugs
- **Then** the prescription is validated, marked `ORDERED`, and formatted as an NCPDP `NewRx` message
- **And** emits an ASTM E2147 audit event `PRESCRIPTION_ISSUED` with clinician NPI and prescription ID.

### Scenario 2: Severe Allergy Cross-Reactivity Intercept (Clinical Edge Case)
- **Given** synthetic patient `p_sarah_connor` has an active allergy record: "Amoxicillin (Severe Anaphylaxis)"
- **When** clinician attempts to prescribe Ampicillin (RxCUI `10582`)
- **Then** the clinical decision support engine detects beta-lactam cross-reactivity
- **And** returns HTTP 422 with alert: `CONTRAINDICATED_ALLERGY_DETECTED`
- **And** order cannot be saved without an override reason and clinical supervisor sign-off.

### Scenario 3: Pharmacy Out-of-Stock and `RxChangeRequest` (Failure Path)
- **Given** an issued `NewRx` prescription sent to synthetic retail pharmacy
- **When** the pharmacy system returns an NCPDP `RxChangeRequest` with code `OUT_OF_STOCK`
- **Then** the prescription status transitions to `CHANGE_REQUESTED`
- **And** an alert is routed to the prescribing clinician's inbox to approve therapeutic substitution.

### Scenario 4: Prescribing Controlled Substance Without MFA Attestation (Misuse Path)
- **Given** an order for Oxycodone (DEA Schedule II)
- **When** a clinician attempts to sign the order without biometric or hardware token 2-factor attestation
- **Then** the EPCS (Electronic Prescriptions for Controlled Substances) validator blocks submission with HTTP 403 Forbidden
- **And** returns error code `EPCS_TWO_FACTOR_REQUIRED`.
