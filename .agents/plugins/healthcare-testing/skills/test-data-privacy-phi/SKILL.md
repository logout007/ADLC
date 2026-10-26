---
name: test-data-privacy-phi
description: Standardized procedure to test for PHI leakage, unauthorized data egress, unmasked log records, and client storage exposure (Skill 4.9).
---

# Skill: Data Privacy & PHI Leakage Testing

Follow this procedure to ensure zero Protected Health Information (PHI) leaks into unapproved channels, telemetry, logs, or unauthorized client roles.

## Pre-conditions
- All sensitive fields must be classified in `docs/spec/` (e.g. `patientName`, `dateOfBirth`, `clinicalNotes`, `mrn`, `ssn`).

## Procedure
1. **Log & Console Scrubbing Verification:**
   - Intercept logger output during request execution.
   - Assert regex patterns for SSN, real names, phone numbers, and clinical note strings NEVER appear in server stdout/stderr.
2. **URL Parameter Sanitization:**
   - Assert endpoints never accept or return PHI in URL query strings (e.g. `/appointments?name=...` $\rightarrow$ Rejected).
   - Only opaque synthetic IDs are permitted.
3. **Response Projection Redaction:**
   - Execute queries with non-clinical roles (`front_desk`, `billing`).
   - Assert sensitive clinical keys (`clinicalNotes`, `symptoms`, `diagnosis`) are completely absent from the JSON response object.
4. **Browser & Client Storage Audit:**
   - Assert zero unencrypted patient data in `localStorage`, `sessionStorage`, or cookies.
5. **Planted Fault Verification:**
   - Intentionally bypass the log scrubber or role projection and assert that the privacy test fails immediately.
