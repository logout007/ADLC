---
name: test-unit-clinical-logic
description: Standardized procedure to implement unit tests for healthcare logic, dosage calculations, and clinical algorithms (Skill 4.1).
---

# Skill: Unit Testing Healthcare Logic

Follow this procedure when creating unit tests for medical logic, dosage calculations, and clinical algorithms.

## Pre-conditions
- Target algorithms must be documented with medical formulas and clinical edge cases in `docs/spec/`.
- Test cases must use synthetic patient demographics.

## Procedure
1. **Define Test Matrix for Medical Calculations:**
   - Test standard adult baselines.
   - Test pediatric boundaries (e.g. weight-based mg/kg dosing).
   - Test geriatric adjustments (e.g. reduced renal clearance / eGFR).
2. **Implement Deterministic Test Suites:**
   - Isolate calculation pure functions from database or network I/O.
   - Verify rounding rules and precision thresholds for medication and lab metrics.
   - Assert expected unit conversions (e.g. mg to mcg, lb to kg, Celsius to Fahrenheit).
3. **Planted Fault Verification:**
   - Plant a mutation in calculation boundary (e.g. `<` to `<=`) and verify tests fail.
4. **Execution:**
   - Run `npm run test` and verify 100% pass rate.
