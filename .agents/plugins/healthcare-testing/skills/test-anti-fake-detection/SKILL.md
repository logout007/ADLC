---
name: test-anti-fake-detection
description: Standardized procedure to detect and reject fake/placebo test cases, trivial assertions, mock poisoning, and mutation-surviving tests using AST linting and planted-fault mutation twins.
---

# Skill: Anti-Fake Test Detection & Mutation Verification

Follow this procedure to mathematically prove that test suites contain **zero fake, placebo, or tautological tests**, and that every test case genuinely verifies functional logic.

## Pre-conditions
1. Test suite is executed under Vitest/Jest with TypeScript.
2. The implementation code under test has accessible export points.

## What Constitutes a "Fake Test"?
A fake test creates the illusion of test coverage while failing to detect bugs:
1. **Assertion-Less Tests:** Tests executing asynchronous functions without `await`, or functions with no `expect()` statements (passes by default if no exception is thrown).
2. **Tautological Assertions:** Assertions that can never fail under any circumstance:
   - `expect(true).toBe(true)`
   - `expect(1).toBe(1)`
   - `expect(typeof result).toBe('object')` (passes even if `result` is `{ error: 'FAILED' }`)
3. **Weak / Trivial Assertions:**
   - `expect(res).toBeDefined()`
   - `expect(res.status).toBeLessThan(500)` (passes on 400 Bad Request when 200 OK was required)
4. **Mock Poisoning:**
   - Mocking the system under test or mocking downstream dependencies to return a static object without validating that inputs were actually processed.
5. **Mutation Survival (The Placebo Test):**
   - The test passes even when the underlying implementation logic is deliberately broken, inverted, or returns dummy values.

## Procedure

### Step 1: Static AST & Assertion Density Linting
1. Inspect test AST (or regex patterns) to verify:
   - Every `it()` or `test()` block contains at least one active assertion.
   - Ban tautologies: `toBe(true)` on boolean literals, `toBeDefined()` without field value checks.
   - Ensure all asynchronous calls are awaited before assertions run.

### Step 2: Mechanical Planted-Fault Mutation Twin (Layer 5)
Every critical functional test suite MUST be subjected to automated mutation testing:
1. **Clean Run:** Run the test suite against pristine code -> Must pass 100%.
2. **Fault Injection:** Automatically inject a mutation into the function under test:
   - Invert a comparison operator (`>` to `<=`, `===` to `!==`).
   - Hardcode a return value to `false`, `null`, or default state.
   - Comment out a state persistence call or audit log write.
3. **Mutant Test Run:** Run the test suite against the mutated code:
   - **REQUIRED OUTCOME:** The test suite MUST FAIL (Mutant is KILLED).
   - **FAILURE CONDITION:** If the test suite passes on the mutated code, **FLAG AS FAKE TEST** (Mutant SURVIVED). The test is untrustworthy and must be rejected.
4. **Code Restoration:** Restore pristine code and verify suite passes again.

### Step 3: CI/CD Enforcement
- Integrate `scripts/detect-fake-tests.js` into the local pre-commit and CI verification pipeline so no PR with fake tests can ever merge.
