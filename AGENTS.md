# AGENTS.md — AI-DLC Standard Operating Rules

This file governs all AI-assisted work in this repository.
**Core Principle:** "The AI writes, machinery enforces, and people decide where it matters."

---

## 1. Principles & Hard Rules
1. **Spec First (with Elicitation):** Every task starts from a written requirement with a testable acceptance criterion in `docs/spec/`. When given a loose prompt, the agent MUST first draft the 4 canonical scenarios using `docs/spec/_SPEC_TEMPLATE.md` with tagged `[CLIENT-DECISION-NEEDED]` flags and obtain human sign-off before writing feature code.
2. **Rules Advise, Machinery Enforces:** Every critical invariant must have a local test, hook, or CI gate behind it (`npm run verify`).
3. **Prove the Checks Work:** Every critical compliance test must have a planted fault it catches and a correct twin it passes.
4. **Stop, Don't Guess:** When the spec is silent, ambiguous, or contradictory, write `HOLD` in `docs/journal.md`, report the issue, and stop. Never invent an enum, schema field, or business logic.
5. **People Own Risky Parts:** Certain architectural and legal areas are human-only and must NEVER be merged by an agent.

---

## 2. Human-Only Areas (Never Agent-Merged)
Changes touching these files/subsystems require named engineer sign-off and cannot be auto-merged:
- **Authentication & Sessions:** Auth tokens, passwords, sessions, MFA.
- **RBAC & Data Projections:** Role permissions, who sees what field (e.g. clinical vs front-desk data).
- **Billing & Payments:** Financial capture, payment processors, charge calculations.
- **Database Migrations:** Any migration running on persistent/production data stores.
- **PHI Allow-List & Vendor Integrations:** Adding endpoints or external services receiving PHI.
- **Clinical Data Storage/Display:** Decisions on what clinical data is persisted or displayed.

---

## 3. Stop Conditions (Write HOLD and Report)
The agent must immediately halt, record a `HOLD` entry in `docs/journal.md`, and output a structured report when:
1. **Spec Silence:** Spec does not define a field name, enum value, or error response.
2. **Spec Contradiction:** Two parts of the spec conflict.
3. **Invariant Test Needs Change:** A compliance or invariant test fails — the implementation is wrong, not the invariant.
4. **Three-Attempt Rule:** 3 attempts to fix an issue have failed.
5. **Contract Variance:** A third-party service differs from recorded fixtures.
6. **Clinical Data Leakage:** A requested change would expose clinical/PHI notes to a non-clinical role.
7. **Architectural Pattern Drift:** Implementation requires a pattern, state strategy, architectural layer, or paradigm not currently established in the codebase.
8. **Code Style & Linter Suppression:** Resolving an issue would require `eslint-disable`, `@ts-ignore`, loose `any` typing, or mass reformatting of untouched legacy code.
9. **Blast Radius Exceeded:** A fix or feature unexpectedly cascades into modifying more than 4 files outside the designated domain/spec boundary.
10. **Unapproved Dependency:** Feature requires introducing a new third-party package, binary, or major version upgrade.
11. **Breaking Contract:** Change breaks backward compatibility of existing public DTOs, database models, or external API contracts.
12. **Unbounded Clinical Query:** Implementation queries clinical datasets without explicit pagination, bounding limits, or indexed column coverage.

### 3.1 HOLD Resume Protocol
When an agent halts on a `HOLD`, it cannot resume until a human engineer adds an explicit resolution entry in `docs/journal.md`:
```markdown
### RESOLVED-HOLD: <INCIDENT-ID>
- Resolution: <Exact architectural decision or schema definition>
- Constraint: <Invariants that must not be altered>
- Status: READY_FOR_AGENT
```
The agent MUST verify this resolution block exists and strictly follow its directive without guessing.

---

## 4. Healthcare Data Guardrails (HIPAA Compliance)
- **Synthetic Only:** Use ONLY synthetic seed records (`p_sarah_connor`, `fd_mark_miller`, `dr_elena_vance`).
- **No Real Data:** Never input real names, real SSNs, real emails, or real clinical notes.
- **PHI Scrubbing:** Loggers, telemetry, and error responses must redact all tagged PHI fields.
- **Zero Frontend Storage Leakage:** Never persist unencrypted PHI in browser `localStorage`, `sessionStorage`, or cookies.
- **Zero PHI in URLs:** Never place patient names, MRNs, DOBs, or clinical conditions in URL query parameters (e.g. `/appointments?name=...`). Use opaque internal UUIDs only.
- **Frontend Telemetry Sanitization:** Error trackers (Sentry/Datadog) must sanitize component state and DOM captures.
- **Audit Logging:** Every access or mutation of patient data must emit an immutable audit event (`timestamp`, `actorId`, `actorRole`, `action`, `resourceId`, `outcome`).

---

## 5. Development Skills Reference
Repeatable development procedures are stored in domain plugins within `.agents/plugins/`:

### Specification & Elicitation Skills:
- `feature-grooming`: Transform raw feature requests, clinical requirements, or user complaints into an implementation-ready specification adhering to `docs/spec/_SPEC_TEMPLATE.md`, the 4 canonical scenarios, and AI-DLC standards.

### Backend (BE) Development Skills:
- `be-add-fhir-endpoint`: Implement FHIR R4 REST APIs (Patient, Encounter, Observation) with US Core profiling.
- `be-add-hl7-processor`: Parse and generate HL7 v2.x (ADT, ORM, ORU) messages with ACK/NAK generation.
- `be-add-clinical-logic`: Implement deterministic medical safety logic (DDI checks, critical lab alerts).
- `be-add-medication-service`: Implement RxNorm medication ordering, allergy cross-reactivity, and e-prescribing.
- `be-add-lab-integration`: Ingest LOINC lab orders and result feeds with abnormal/critical threshold routing.
- `be-add-scheduling`: Build appointment availability, calendar booking, and transactional conflict detection.
- `be-add-encryption`: Implement field-level AES-256-GCM encryption for sensitive PHI (SSN, payment tokens).
- `be-add-audit-pipeline`: Implement immutable HIPAA audit trail logging with tamper-evident SHA-256 hashing.
- `add-endpoint`: Implement authenticated API routes with RBAC projection and consent validation.
- `add-adapter`: Integrate external systems (EHR/FHIR/LIS) via Anti-Corruption Layer (ACL).
- `add-notification`: Implement HIPAA-compliant messaging (zero PHI on unencrypted channels).

### Frontend (FE) Development Skills:
- `fe-add-clinical-components`: Build reusable clinical UI components (Patient Card, Medication List, Lab Table).
- `fe-add-clinical-view`: Build accessible clinical dashboards (vitals, labs, problems) with WCAG 2.1 AA and print CSS.
- `fe-add-phi-form`: Build HIPAA-compliant intake and clinical documentation forms with zero client storage leakage.
- `fe-add-search-filter`: Implement patient search and clinical filtering interfaces with autocomplete and MRN lookup.
- `fe-add-break-glass-modal`: Build emergency clinical access override UI with mandatory justification capture.
- `fe-add-realtime-alerts`: Implement WebSocket connections for stat clinical alerts and sound/visual badges.
- `fe-add-auth-ui`: Build secure healthcare login, MFA input, and automatic 15-minute inactivity session lock.
- `fe-add-abdm-flow`: Implement National Health ID (ABHA/ABDM) enrollment, Aadhaar dual-OTP checks, holder diffing, and consent management.
- `fe-add-audit-inspector`: Implement client-side ASTM E2147-18 tamper-evident chained audit logging and verification inspection modal.

### Database & Performance Skills:
- `add-migration`: Draft backward-compatible healthcare database schema updates with rollback scripts.
- `db-add-query-optimization`: Implement clinical database indexing (MRN, DOB, encounter timelines) and lean queries.

### Testing & QA Engineering Skills:
- `test-unit-clinical-logic`: Implement unit tests for healthcare logic, dosage calculations, and clinical algorithms (Skill 4.1).
- `test-integration-systems`: Test integrations between healthcare components, EHR adapters, and databases (Skill 4.2).
- `test-clinical-validation`: Validate clinical accuracy, drug interactions, and medical contraindication rules (Skill 4.3).
- `test-security-access`: Test authentication, RBAC, ABAC, and privilege escalation vulnerabilities (Skill 4.4).
- `test-api-fhir-conformance`: Test RESTful APIs and US Core FHIR R4 conformance, pagination, and error responses (Skill 4.5).
- `test-performance-scale`: Test performance, concurrency, response time thresholds, and DB query efficiency under load (Skill 4.6).
- `test-accessibility-wcag`: Test clinical user interfaces for WCAG 2.1 AA accessibility and assistive technology (Skill 4.7).
- `test-mobile-apps`: Test mobile patient portals and clinician mobile rounding apps on device boundaries (Skill 4.8).
- `test-data-privacy-phi`: Test for PHI leakage, unauthorized data egress, unmasked log records, and client storage (Skill 4.9).
- `test-anti-fake-detection`: Detect and reject fake/placebo tests, trivial assertions, and mock poisoning using AST linting and mutation twins.
- `test-auto-generation-grooming`: Synthesize and groom functional test cases with common-sense criteria without duplicate bloat.
- `test-functional-e2e`: Construct and execute stateful End-to-End multi-step user and functional journeys across complete system lifecycles.
- `write-invariant-test`: Write healthcare compliance invariant tests with planted-fault verification twins under Layer 4 and Layer 5.
- `qa-adversarial-review`: Mechanically audit Pull Requests for bypassed data projections, missing audit logs, and race conditions before merge.

## 6. Lifecycle Role Division
- **Developer / AI Engineer (This Workspace):** Owns feature implementation against written specs, executes development skills, enforces PHI scrubbing and audit trails, halts on Stop Conditions, and flags Human-Only PRs.
- **QA Lead & Test Engineering (Separate Track):** Owns the invariant test suites, planted-fault verification harness, persona seeds, and E2E regression runs.

### 6.1 AI Review Safeguards (Machinery Before LLM)
- **Deterministic Linters First:** Before any AI reviewer evaluates a PR, automated linters (ESLint security rules, Semgrep, strict type checking) must run cleanly. An LLM must never replace mechanical verification.
- **Adversarial Review Directive:** When an AI review agent is invoked, its prompt must be strictly adversarial: instructed to hunt for bypassed data projections, missing audit records, unmasked URLs, and asynchronous race conditions, rather than politely approving.

---

## 7. Definition of Done Checklist
Before marking any task complete:
- [ ] `npm run verify` passes with 0 skipped or disabled tests.
- [ ] Reverting the change causes at least one test to fail.
- [ ] No undeclared enums, error codes, or config keys added.
- [ ] No PHI field reaches logs, snapshots, or unauthorized response bodies.
- [ ] Every patient data access generates an audit record.
- [ ] Traceability matrix in `docs/traceability.md` is updated.
- [ ] Any human correction codified into a rule or test in `docs/journal.md`.
- [ ] PR template completed with the "Not Checked" section filled out.

---

## 8. Continuous Learning & Feedback Loop (Layer 10)
- **Every Correction Becomes a Rule or Test:** When an agent error, human review correction, or invariant failure occurs, record a `CORRECTION` entry in `docs/journal.md`.
- **Codification Requirement:** Never conclude a corrected task by simply patching code; you must either (1) add a planted-fault invariant test to prevent regression, or (2) add an explicit rule in `AGENTS.md` or the corresponding `SKILL.md`.
