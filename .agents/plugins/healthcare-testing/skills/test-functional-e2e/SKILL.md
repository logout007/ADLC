---
name: test-functional-e2e
description: Standardized procedure to construct and execute stateful End-to-End (E2E) multi-step user and functional journeys across complete system lifecycles.
---

# Skill: Stateful Functional End-to-End (E2E) Testing

Follow this procedure when constructing End-to-End tests that validate complete functional lifecycles across multiple dependent steps, persistence, state transitions, and asynchronous side effects.

## Pre-conditions
1. All services, APIs, and databases are runnable in an isolated in-memory or ephemeral test harness.
2. Synthetic test personas are used for all actor roles (`test/fixtures/personas.ts`).
3. State is reset before each E2E test execution (`beforeEach`).

## Procedure

### Step 1: Map the Multi-Step Lifecycle Journey
E2E testing differs from unit and integration testing because it validates **sequential state evolution**:
1. **Initial Step (Creation / Ingestion):**
   - The user or external system triggers an entity creation (e.g. Recording initial intake vitals or booking an appointment).
   - Assert HTTP status code, response body, and persistence in storage.
2. **Intermediate Step (State Mutation & Boundary Check):**
   - Perform an action that mutates the entity or triggers a conditional branch (e.g. Recording critical abnormal vitals or attempting a conflicting schedule).
   - Assert that downstream escalation rules, conflict blocks, or state transitions fire correctly.
3. **Multi-Role Step (Cross-Actor Handoff):**
   - A different role or consumer queries or acts on the modified entity (e.g. Attending clinician reviews alerts flagged by intake nurse).
   - Assert role-specific data boundaries, projection views, and access policies.
4. **Final Step (Resolution / Audit Integrity):**
   - Entity reaches terminal state (e.g. Discharged, Resolved, or Cancelled).
   - Assert that audit trail has recorded every chronological step with cryptographic link integrity.

### Step 2: Ensure Deterministic Isolation
1. **Zero Cross-Test Pollution:** Every E2E test must operate on freshly reset state.
2. **Explicit Teardown:** Clean up database rows, audit records, and mock timers in `afterEach`.
3. **No Flaky Timing:** Never rely on fixed sleeps; wait on promises or deterministic completion events.

### Step 3: Verification & Execution
- Run `npm run test:e2e` to execute the full functional E2E test suite.
- Verify that every step in the journey passed without shortcutting assertions.
