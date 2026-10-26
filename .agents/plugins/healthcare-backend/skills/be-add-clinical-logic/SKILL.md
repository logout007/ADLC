---
name: be-add-clinical-logic
description: Standardized procedure to implement deterministic clinical business logic (drug-drug interactions, critical lab value routing, pediatric dosage limits).
---

# Skill: Backend - Add Clinical Decision Support (CDS) Logic

Follow this procedure whenever implementing clinical decision support (CDS) algorithms, medical safety rules, or dosage calculations.

## Safety & Medical Regulatory Principles
- **FDA Software as a Medical Device (SaMD) & IEC 62304:** Clinical logic engines must be deterministic, isolated from database side effects, and mathematically verified.
- **ISO 14971 Medical Device Risk Management:** Any potential for fatal clinical consequences (e.g. cardiac arrest from undetected hyperkalemia, anaphylaxis from penicillin cross-reactivity) must have risk mitigation built into the software architecture.
- **Fail-Secure Default:** When input data is ambiguous or interaction databases are unreachable, the system must fail-secure (alerting the clinician rather than silently proceeding).

## Implementation Steps

### 1. Spec Alignment
- Confirm the rule in `docs/spec/` (e.g., `INV-MED-001`, `INV-LAB-001`).
- Verify exact threshold values, units (UCUM), and severity classification (`CONTRAINDICATED`, `MAJOR`, `MODERATE`).

### 2. Implement Pure Evaluation Functions
- Create service module in `src/services/clinical/`:
  - `evaluateDrugInteractions(newRx, currentMedications, allergies): DurAlert[]`
  - `evaluateLabCriticalThreshold(loincCode, value, ucumUnit, patientAge): CriticalAlert | null`
- Pure function requirement: No network calls or database queries inside the calculation core.

### 3. Handle Alert Escalation
- If a check triggers:
  - Return structured clinical alert DTO with severity, clinical rationale, and required override level.
  - If critical threshold (e.g. Potassium $> 6.2\text{ mmol/L}$), dispatch real-time stat escalation cascade.

### 4. Audit Logging (ASTM E2147-18)
- Emit an immutable audit event for every clinical alert and clinician override:
  - `action`: `CLINICAL_SAFETY_ALERT_TRIGGERED` | `ALERT_OVERRIDDEN`
  - `actorId`: Prescribing clinician
  - `severity`: Alert severity
  - `overrideReason`: Captured override justification text (if overridden)

### 5. Stop Conditions
- **HOLD:** If the spec lacks reference ranges for specific demographic groups (e.g. pediatric ranges).
- **HOLD:** If the clinical override workflow is undefined for life-threatening drug interactions.
