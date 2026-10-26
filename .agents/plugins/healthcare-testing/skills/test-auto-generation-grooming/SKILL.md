---
name: test-auto-generation-grooming
description: Standardized procedure to automatically synthesize and groom functional test cases with common-sense criteria (eliminating duplicate bloat, tautologies, flaky delays) when new functionality is introduced.
---

# Skill: Automated Functional Test Generation & Common-Sense Grooming

Follow this procedure whenever a new functionality, service method, or API endpoint is added to automatically generate genuine, groomed functional test suites without manual guesswork.

## Pre-conditions
1. The new functionality has a defined interface, input schema, output contract, and error conditions.
2. Synthetic test personas and data factories are available (`test/fixtures/personas.ts`).
3. Pure functional logic is isolated from non-deterministic external side effects.

## Procedure

### Step 1: Automated Contract Decomposition
When a new function or endpoint is implemented, parse its signature and decompose it into 4 mandatory test dimensions:
1. **Happy Path (Positive Contract):**
   - Valid inputs yield expected result and valid state mutations.
   - Example: Normal vital sign inputs classify status as `NORMAL` and persist an observation record.
2. **Boundary Value Analysis (BVA):**
   - Test exact numerical or length thresholds (`min`, `min - 1`, `max`, `max + 1`, zero, empty string, maximum array capacity).
   - Example: SpO2 threshold at 88% is `ABNORMAL`, while 87% escalates to `CRITICAL`.
3. **Negative / Rejection Paths:**
   - Missing required fields, out-of-bounds biological ranges, unauthorized actors, malformed formats.
   - Must assert exact error codes (e.g. `INVALID_VALUE`, `OUT_OF_RANGE`), not just generic 400s.
4. **State Transition & Concurrency:**
   - Validating state transitions (e.g. `PENDING` -> `REVIEWED` -> `ARCHIVED`) and rejecting invalid state jumps.

### Step 2: "Common-Sense" Grooming Rules
Automatically groom the generated test suite before committing:
- **Rule 1: Eliminate Bloat & Redundancy:**
  - If a schema linter or Zod already verifies string types, do not write 15 tests checking character lengths. One boundary test is sufficient.
- **Rule 2: Strip Tautological Assertions:**
  - Reject tests where assertions only check that the variable exists (e.g. `expect(res).toBeDefined()`) or tests comparing hardcoded constants (`expect(true).toBe(true)`).
- **Rule 3: Ban Mock Poisoning:**
  - Never mock the function under test. Only mock external network calls or third-party gateways. Mocks must reflect real service contracts.
- **Rule 4: Flake-Free Assertions:**
  - Ban `setTimeout` or arbitrary sleep delays. Use event-driven await hooks or polling predicates with strict timeouts.
- **Rule 5: Semantic Readability (AAA Pattern):**
  - Arrange: Setup synthetic fixtures.
  - Act: Execute the function under test once.
  - Assert: Verify specific return value AND persistent state side effects.

### Step 3: Verification & Execution
1. Run `npx vitest run <generated-test-file>`.
2. Verify all generated and groomed test cases pass 100%.
3. Pass the generated suite to `test-anti-fake-detection` to ensure no fake tests exist.
