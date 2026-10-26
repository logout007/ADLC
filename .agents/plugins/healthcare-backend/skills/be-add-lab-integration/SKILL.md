---
name: be-add-lab-integration
description: Standardized procedure to implement laboratory order entry, LOINC-coded result processing, critical value alerting, and result trending.
---

# Skill: Backend - Lab Orders & Result Processing

Follow this procedure whenever implementing laboratory orders, LIS result feeds, or diagnostic observations.

## Principles
1. **LOINC Standardization:** All lab tests must be mapped to valid LOINC codes.
2. **Critical Value Routing:** Any result exceeding critical thresholds (e.g. Potassium $> 6.2\text{ mmol/L}$) must trigger real-time escalation.
3. **Result Immutability:** Once marked `FINAL`, results cannot be deleted. Changes must be posted as `AMENDED` or `CORRECTED` with reason.

## Implementation Steps

### 1. Spec Alignment
- Confirm lab test codes, normal reference ranges, and critical thresholds in `docs/spec/04_lab_results_vitals_observations.md`.

### 2. Implement Order Entry
- Validate order: `patientId`, `loincCode`, `testName`, `specimenType`, `priority` (`STAT` | `ROUTINE`).
- Persist order with status `ORDERED`.

### 3. Implement Result Ingestion & Normalization
- Ingest result from LIS (HL7 ORU or JSON webhook).
- Extract: `value`, `ucumUnit`, `referenceRangeLow`, `referenceRangeHigh`, `resultTimestamp`.
- Compare value against reference range:
  - If inside range: `interpretation = 'NORMAL'`.
  - If outside range but below critical threshold: `interpretation = 'ABNORMAL'`.
  - If exceeds critical threshold: `interpretation = 'CRITICAL'`.

### 4. Critical Alert Dispatch
- If `interpretation === 'CRITICAL'`:
  - Trigger high-priority clinician notification (SMS/Push/In-app).
  - Record audit log: `action: CRITICAL_LAB_VALUE_TRIGGERED`.

### 5. Stop Conditions
- **HOLD:** If an incoming lab test lacks age-adjusted or sex-adjusted reference ranges in the spec.
