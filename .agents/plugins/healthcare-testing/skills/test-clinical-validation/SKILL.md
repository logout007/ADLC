---
name: test-clinical-validation
description: Standardized procedure to validate clinical accuracy, drug interactions, and medical contraindication rules against clinical standards (Skill 4.3).
---

# Skill: Clinical Validation Testing

Follow this procedure to validate medical rules, drug-drug interaction (DDI) logic, and allergy cross-reactivity.

## Pre-conditions
- Target drug interactions and allergy matrices must be coded with RxNorm and SNOMED CT in `docs/spec/`.
- Medical rules must follow established clinical decision guidelines (e.g. DUR standards).

## Procedure
1. **Define Clinical Matrix Scenarios:**
   - Level 1 (`CONTRAINDICATED`): e.g. Sildenafil + Nitroglycerin $\rightarrow$ Hard stop.
   - Level 2 (`MAJOR`): e.g. Warfarin + NSAID $\rightarrow$ Bleeding risk warning.
   - Allergy Cross-Reactivity: e.g. Penicillin anaphylaxis + 1st gen Cephalosporin $\rightarrow$ Interruptive warning.
2. **Implement Validation Tests:**
   - Feed synthetic clinical profiles into the validation engine.
   - Assert exact warning severity levels and clinical recommendations.
   - Verify that non-fatal warnings permit override only with signed clinical justification.
3. **Planted Fault Verification:**
   - Invert a contraindication check (e.g. allowing Level 1 without stop) and ensure the test suite fails immediately.
