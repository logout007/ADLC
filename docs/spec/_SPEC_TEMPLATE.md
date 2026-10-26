# Spec Template & Elicitation Protocol
## Document: `docs/spec/NN_<feature_name>.md`

> **Usage:** Whenever a client provides a loose or informal request (e.g. *"We need a doctor list of appointments page"*), the AI Agent MUST NOT write code immediately. Instead, copy this template, draft all 4 canonical scenarios, flag any ambiguities with `[CLIENT-DECISION-NEEDED]`, and obtain human sign-off before proceeding.

---

# Domain Specification: [Feature Name]

## 1. Feature Intent & Clinical Objective
- **Goal:** [High-level summary of the clinical or administrative workflow]
- **Target Persona:** [Primary persona, e.g. `dr_elena_vance`, `p_sarah_connor`, `fd_mark_miller`]
- **Healthcare Standards:** [US Core FHIR R4 / LOINC / RxNorm / NCPDP / ASTM E2147-18]

---

## 2. Access Matrix & Data Projection
| Persona Role | Permitted Fields | Forbidden / Redacted Fields |
|---|---|---|
| `dr_elena_vance` (Clinician) | Clinical notes, chief complaint, Rx history | Billing token, SSN |
| `fd_mark_miller` (Front Desk) | Appointment time, provider, status | Clinical notes, diagnosis codes |
| `p_sarah_connor` (Patient) | Own demographic info, scheduled time | Other patients' records, clinician private notes |

---

## 3. The 4 Canonical Scenarios

### Scenario 1: Happy Path
- **Given:** [Valid synthetic state, authenticated persona, required consents on file]
- **When:** [Standard user action or API request occurs]
- **Then:** [Expected successful state change, UI rendering, and audit log emitted]

### Scenario 2: Edge Case
- **Given:** [Boundary condition, e.g. Daylight Saving Time timezone shift, cross-midnight shift, 0 records found]
- **When:** [Request is dispatched]
- **Then:** [System handles gracefully without crash, pagination or empty state rendered safely]

### Scenario 3: Failure Path
- **Given:** [Downstream service unavailable, network timeout, database lock]
- **When:** [Request is processed]
- **Then:** [Sanitized error code returned (`ERR_SCHEDULE_UNAVAILABLE`), zero stack traces or PHI leaked]

### Scenario 4: Misuse / Security Path
- **Given:** [Unauthorized persona attempts action OR invalid/crafted payload]
- **When:** [Request hits the API endpoint]
- **Then:** [HTTP 403 Forbidden or 422 Unprocessable Entity, access attempt logged in audit trail]

---

## 4. Open Decisions & Gaps List (Client Sign-Off Required)
- `[CLIENT-DECISION-NEEDED: 01]`: [e.g. Are cancelled appointments shown in grey or hidden by default?]
- `[CLIENT-DECISION-NEEDED: 02]`: [e.g. What is the cancellation window cutoff before marking as a No-Show?]

---

## 5. Definition of Done Checklist for this Spec
- [ ] Spec reviewed and signed off by Human Lead.
- [ ] No undeclared enums or fields introduced.
- [ ] Zero PHI exposed in URL query parameters or client storage.
- [ ] Audit event emitted on every read and write.
