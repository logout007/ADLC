---
name: be-add-medication-service
description: Standardized procedure to implement medication ordering, RxNorm code validation, drug-drug interaction (DDI) checking, and allergy cross-reactivity rules.
---

# Skill: Backend - Medication Management & Safety

Follow this procedure whenever implementing medication ordering, prescription management, or medication reconciliation.

## Clinical Safety & Industry Standards
- **Standardized Terminology:** RxNorm Concept Unique Identifier (RxCUI) for clinical drugs and National Drug Code (NDC) for package formulations.
- **E-Prescribing Standard:** **NCPDP SCRIPT Standard v2017071** (`NewRx`, `RxChangeRequest`, `RxChangeResponse`, `CancelRx`).
- **Drug Utilization Review (DUR):** Severity levels: `Level 1: CONTRAINDICATED` (hard stop), `Level 2: MAJOR` (interruptive alert), `Level 3: MODERATE`, `Level 4: MINOR`.
- **DEA EPCS Safeguards:** Electronic Prescriptions for Controlled Substances (Schedule II-V) mandate two-factor clinician attestation and zero automated refills on Schedule II drugs.

## Implementation Steps

### 1. Spec Verification
- Confirm medication rules and severity thresholds in `docs/spec/03_medication_reconciliation_allergies.md`.

### 2. Implement Order Validation Engine
- Validate order fields: `rxCui`, `drugName`, `dose`, `route`, `frequency`, `quantity`, `refillsAllowed`.
- Enforce Schedule II rules (`refillsAllowed === 0`).

### 3. Run Safety Interceptors
- **Drug-Allergy Cross-Reactivity:**
  - Map patient allergies to pharmacological classes (e.g. Penicillin $\rightarrow$ Beta-lactam).
  - Flag potential cross-reactivity with the ordered medication.
- **Drug-Drug Interactions (DDI):**
  - Compare new Rx against all `status: 'ACTIVE'` medications in patient profile.
  - Return DUR severity classification.

### 4. Persistence, NCPDP Message Formatting & Audit
- Persist order with status `ORDERED`.
- Format outbound message as NCPDP SCRIPT v2017071 `NewRx` XML payload.
- Emit ASTM E2147 audit event:
  - `action`: `PRESCRIPTION_ORDER_CREATED`
  - `actorId`: Prescribing clinician NPI
  - `resourceId`: `prescriptionId`
  - `durAlertCount`: Count of alerts detected and overridden

### 5. Stop Conditions
- **HOLD:** If the spec does not define the override procedure for Level 1 Contraindicated medications.
