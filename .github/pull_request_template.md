## Description of Changes
<!-- Provide a concise summary of what was changed and why -->

## Linked Requirement / Milestone
- Spec clause: `docs/spec/` (e.g. SPEC-001, INV-001)
- Milestone / Issue: #

## Definition of Done Verification
- [ ] `npm run verify` passed locally with zero warnings or failures.
- [ ] At least one test covers this change and fails if the change is reverted.
- [ ] No new enum value, error code, or config key was added unless defined in `docs/spec/`.
- [ ] No PHI field reaches logs, snapshots, or unauthorized client responses.
- [ ] Zero PHI in URL query parameters, unencrypted browser storage, or client error telemetry.
- [ ] Every patient data access or modification generates an audit event.
- [ ] Synthetic personas only — no production data or credentials used.
- [ ] Deterministic linters (ESLint, Semgrep, strict TypeScript) passed before AI review.
- [ ] Planted fault test passed (`npm run test:planted-fault`).

## Human-Only Areas Touched
Does this PR touch any of the following? (Check all that apply):
- [ ] Authentication, sessions, MFA
- [ ] Role-based access control or field-level data projections
- [ ] Billing, payments, fee calculations
- [ ] Database schema migrations on real data
- [ ] PHI allow-list or third-party vendor integrations
- [ ] Storage or display of clinical data

*Note: If any box above is checked, this PR CANNOT be merged by an AI agent. It requires manual review and approval by a named human engineer.*

---

## What Was NOT Checked (Mandatory Section)
<!-- Be completely honest and specific. List everything that was NOT exercised, automated, or verified in this run. Examples: Edge cases with latency > 5s, IE11 browser compatibility, load testing over 500 RPS, etc. -->
- 
- 
