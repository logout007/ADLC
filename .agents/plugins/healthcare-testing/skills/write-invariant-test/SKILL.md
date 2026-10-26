---
name: write-invariant-test
description: Standardized procedure to write healthcare compliance invariant tests with planted-fault verification twins under Layer 4 and Layer 5 of AI-DLC.
---

# Skill: Write Invariant Test

Follow this procedure whenever adding or modifying compliance invariant tests in `test/invariants/`.

## Pre-conditions
1. Invariant must be documented in `docs/spec/` with an explicit identifier (e.g. `INV-001`, `INV-002`, `INV-003`).
2. Target personas must be synthetic records defined in `test/fixtures/personas.ts`.
3. Invariant must NEVER be altered to make failing feature code pass (Principle 2 & Stop Condition 3).

## Procedure

### Step 1: Invariant Specification & Assertion Definition
- Map the invariant requirement directly to a test suite:
  - **INV-001 (Consent Gate):** Booking or clinical action MUST fail with HTTP 403 / `CONSENT_REQUIRED` if valid consent is not timestamped prior to the action.
  - **INV-002 (RBAC Redaction / Minimum Necessary):** The outbound response for non-clinical roles (e.g. `front_desk`) MUST NEVER contain sensitive clinical keys or values (e.g. `clinicalNotes`, `diagnosis`).
  - **INV-003 (Immutable Audit Trail):** Every data access or mutation MUST emit an audit event with `{ timestamp, actorId, actorRole, action, resourceId, outcome }`.

### Step 2: Persona-Driven Test Implementation
- Construct tests against the 4 canonical scenario types:
  1. **Happy Path:** Valid persona performing permitted action $\rightarrow$ Success (200/201) + Audit Log.
  2. **Edge Case:** Expired or revoked consent $\rightarrow$ Rejection (403).
  3. **Failure Path:** Resource conflict or double-booking $\rightarrow$ Conflict (409) with zero leaked PHI.
  4. **Misuse Path:** Unauthorized role attempting to read restricted fields $\rightarrow$ Redacted response or 403 Forbidden.

### Step 3: Planted-Fault Twin Verification (Layer 5)
- Every invariant test MUST have a corresponding planted fault in `scripts/test-planted-fault.js`:
  1. The planted fault intentionally introduces the exact bug the invariant guards against (e.g. disabling the redaction filter or bypassing the consent check).
  2. Run the test against the planted fault $\rightarrow$ Verify the invariant test FAILS (Exit Code $\neq 0$).
  3. Revert to correct code $\rightarrow$ Verify the invariant test PASSES (Exit Code $0$).
- If the test does NOT fail on the planted fault, the test is invalid and must be rewritten.

### Step 4: Verification & Traceability
- Run `npm run verify` to ensure all invariant tests run and pass.
- Update `docs/traceability.md` mapping the spec invariant ID to the test file.
