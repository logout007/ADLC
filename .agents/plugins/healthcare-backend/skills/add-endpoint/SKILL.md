---
name: add-endpoint
description: Standardized procedure to implement an authenticated healthcare endpoint with RBAC projection, consent validation, and audit logging.
---

# Skill: Add Endpoint

Follow this step-by-step procedure whenever adding or modifying an HTTP endpoint.

## Pre-conditions
1. Confirm the endpoint requirement exists in `docs/spec/` with acceptance criteria and persona scenarios.
2. Confirm role permissions and field-level projections are documented.

## Implementation Steps
1. **Define Types & Schema:**
   - Define request payload and response projection DTOs.
   - Tag all PHI fields (e.g. `medicalNotes`, `diagnosis`, `patientName`).
2. **Implement Route Handler:**
   - Attach authentication and role extraction middleware.
   - Check patient consent before performing clinical or appointment actions.
   - Apply role-based projection (e.g. strip `clinicalNotes` for `front_desk` role).
3. **Emit Audit Event:**
   - Record `actorId`, `actorRole`, `action`, `resourceId`, and `outcome`.
4. **Scrub Output:**
   - Ensure logger only logs non-PHI identifiers and status codes.
5. **Update Traceability:**
   - Map route to spec requirement in `docs/traceability.md`.
6. **Verify:**
   - Run `npm run verify` to ensure tests and PHI linters pass.
