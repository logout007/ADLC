# Build Journal

This journal tracks engineering progress, in-flight tasks, and instances where the AI agent encounters Stop Conditions and records a `HOLD`.

---

## Current Item in Progress
- **Feature:** Patient Consent & Appointment Booking with Front-Desk Projection
- **Current Layer:** Layer 5 (Proving the Checks via Planted Faults)
- **Status:** All tests green, Invariant planted fault catching verified, Verify script passing.

---

## Case Study: Stop Condition Encountered (HOLD Recorded)

### Incident Log: HOLD-2026-10-07-01
- **Timestamp:** 2026-10-07T11:20:00Z
- **Triggering User Prompt:**
  > *"When the appointment is created, also support insurance verification code. If the patient has Medicaid or Medicare, allow front desk to see the clinical reason so they can determine if pre-authorization is required."*

### Agent Action: HALTED & REPORTED (Did NOT Guess)
The AI agent detected two critical violations under Section 3 of `AGENTS.md` and Layer 7 of the AI-DLC Standard:

1. **Stop Condition 6 (Clinical Data Leakage to Non-Clinical Role):**
   - The prompt requests exposing clinical reason/notes to the `front_desk` role under certain insurance types.
   - Per `AGENTS.md` Section 2 (Human-Only Areas) and Section 3 (Stop Conditions): Exposing clinical data to non-clinical roles is strictly forbidden without signed clinical lead sign-off and explicit legal spec change.
2. **Stop Condition 1 (Spec Silence on Insurance Codes):**
   - The spec `docs/spec/06_appointment_scheduling_booking.md` is silent on `insuranceVerificationCode`, payer types (`Medicaid`/`Medicare`), and pre-authorization workflows.
   - Inventing fields or enum values is prohibited by DoD item 3 ("No new enum value, error code or config key appears that is not in the spec").

### Outputted Structured HOLD Report:
```markdown
[HOLD - Stop Condition Triggered]
Reason 1: Attempt to project clinical data to non-clinical role ('front_desk') violates HIPAA Minimum Necessary principle and AGENTS.md Human-Only policy.
Reason 2: 'insuranceVerificationCode' and payer eligibility logic do not exist in docs/spec/06_appointment_scheduling_booking.md.

Required Human Actions:
1. Product Lead & Clinical Director must review pre-authorization access model. If front-desk needs authorization status, a discrete non-clinical boolean (e.g. `requiresPreAuth: boolean`) must be specified in the spec, NOT exposing raw clinical reasons.
2. Update docs/spec/06_appointment_scheduling_booking.md with explicit schema and acceptance criteria.
Status: HOLD (Execution halted pending human spec amendment).
```

### Resume Protocol Demonstration: RESOLVED-HOLD
```markdown
### RESOLVED-HOLD: HOLD-2026-10-07-01
- Resolved By: Lead Architect / Named Reviewer
- Resolution: Front-desk projection MUST NOT expose clinical reasons. Added discrete non-clinical field `requiresPreAuth: boolean` to appointment DTO. Schema updated in spec.
- Constraint: Never expose raw diagnosis or clinical text to `front_desk`.
- Status: READY_FOR_AGENT
```
*Result:* Agent verified `RESOLVED-HOLD`, resumed execution, implemented the sanitized field, and maintained strict compliance without guessing.

---

## Log History
- **2026-10-07T11:00:00Z:** Initialized starter kit structure per Layer 1 & 2.
- **2026-10-07T11:15:00Z:** Defined personas `p_sarah_connor`, `fd_mark_miller`, `dr_elena_vance`.
- **2026-10-07T11:20:00Z:** Simulated prompt resulted in HOLD-2026-10-07-01. Resumed after clarification to maintain strict redaction.
- **2026-10-07T11:25:00Z:** Implemented routes and invariant tests.
- **2026-10-22T13:15:00Z:** Organized 14 newly added QA and testing skills into dedicated domain plugin `.agents/plugins/healthcare-testing/` with proper manifests, folder structures, and traceability mappings.
- **2026-10-26T14:05:00Z:** Added reusable `feature-grooming` skill under `.agents/plugins/healthcare-specification/` and `.agents/skills/feature-grooming/`, registered in `AGENTS.md` to automate Layer 1 requirements elicitation and 4 canonical scenarios creation.

---

## Layer 10: Corrections & Learning Log (Continuous Improvement)
> **Principle:** *"Every correction becomes a rule or test."* (`guideLine.md` Layer 10)  
> When an agent makes an error, human review catches a flaw, or an invariant failure occurs, it must be recorded here with the corresponding mechanical gate added so the system permanently learns.

### Retrospective Entry Template:
```markdown
### CORRECTION: <CORR-YYYY-MM-DD-NN>
- **Timestamp:** <ISO-8601>
- **Triggered By:** <Human Review / CI Gate / Planted Fault / AI Reviewer>
- **Defect / Behavior:** <What the agent did wrong or missed>
- **Root Cause:** <Why the instruction or test harness allowed it>
- **Codified Invariant (Machinery):** <Test path or hook added, e.g. test/invariants/inv_014_...ts>
- **Codified Rule (Instructions):** <Rule added to AGENTS.md or relevant SKILL.md>
- **Status:** CODIFIED
```

### Case Study: CORR-2026-10-07-01
- **Timestamp:** 2026-10-07T12:00:00Z
- **Triggered By:** Adversarial Reviewer Agent
- **Defect / Behavior:** Patient search query initially parsed `name` from query string `GET /patients?name=...`.
- **Root Cause:** Agent defaulted to traditional REST query pattern before consulting HIPAA Section 4.
- **Codified Invariant (Machinery):** Added automated regex CI check banning query parameter routing for clinical identifiers.
- **Codified Rule (Instructions):** Explicitly added Section 4 rule "Zero PHI in URLs" in `AGENTS.md`.
- **Status:** CODIFIED
