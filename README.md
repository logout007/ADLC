# ADLC

A spec-driven, machinery-enforced AI software development lifecycle (AI-DLC) for building regulated, HIPAA-compliant healthcare software with coding agents.

**Topics:** `ai-agents` · `sdlc` · `healthcare` · `claude-code` · `hipaa` · `fhir` · `spec-driven-development`

---

## Problem Statement

Coding agents write code fast, but speed is the wrong axis for regulated software. In healthcare, a plausible-looking but unchecked change can leak PHI, expose clinical notes to a front-desk role, or invent a schema field that becomes a legal liability. The failure modes are consistent:

- **Loose prompts invite guessing.** Given "add insurance verification to bookings," an agent will happily invent enums, field names, and access rules that nobody specified.
- **Rules alone don't bind.** Prose instructions ("don't leak PHI") are advisory; without a test, hook, or CI gate behind them, they are silently ignored under pressure.
- **Unproven checks give false confidence.** A green test suite is worthless if the test would also pass against a broken implementation.
- **Nobody knows what was *not* checked.** "It works" claims hide the risky corners — auth, RBAC projections, billing, migrations — that need a human.

**ADLC is the operating system for trustworthy agent work.** Its principle: *"The AI writes, machinery enforces, and people decide where it matters."* This repository is the reference starter kit — rules (`AGENTS.md`), the ten-layer standard (`guideLine.md`), repeatable domain skills (`.agents/`), specifications (`docs/spec/`), and the traceability / decision / journal records that make agent output auditable.

---

## Task Decomposition

Complex work is never handed to an agent as one big prompt. It is decomposed through the ten-layer lifecycle into small, individually verifiable units:

1. **Intent → Spec (Layer 1).** Every request is first rewritten as a written requirement with a *testable* acceptance criterion. No spec, no code.
2. **Spec → 4 Canonical Scenarios.** Using `docs/spec/_SPEC_TEMPLATE.md`, each feature is split into the four scenarios that define its blast radius: **happy path**, **edge case**, **failure path**, and **misuse / security path**. Ambiguities are tagged `[CLIENT-DECISION-NEEDED]` for human sign-off before any code is written.
3. **Spec → Access Matrix.** A per-role "who sees which fields" table is fixed up front (clinician vs front-desk vs patient), so data-projection rules are decided, not discovered mid-implementation.
4. **Scenarios → Skills (see below).** Each scenario maps to a small, single-responsibility skill that touches one domain boundary.
5. **Skill → Proof (Layers 4–5).** Each unit is backed by a compliance test with a **planted fault** it must catch and a **correct twin** it must pass.
6. **One item in progress.** `docs/journal.md` tracks a single active work item; the agent halts with a `HOLD` rather than parallelizing into unbounded change.

The decomposition stops being "one feature" and becomes a set of independently spec'd, independently tested, independently reviewable slices — each small enough that a mistake is contained.

---

## Skill Selection

Skills in `.agents/` are the repeatable procedures each decomposed task runs through. Selection is driven by the **domain spec ↔ skill mapping** in [`docs/traceability.md`](docs/traceability.md), not by ad-hoc choice:

- **Start with `feature-grooming`** whenever the input is a loose prompt — it produces the spec and 4 scenarios before anything else.
- **Match the layer of the change:**
  - *Backend/interop* → `be-add-fhir-endpoint`, `be-add-hl7-processor`, `be-add-scheduling`, `be-add-medication-service`, `be-add-lab-integration`, `add-adapter`, `add-endpoint`, `add-notification`, `be-add-encryption`, `be-add-audit-pipeline`.
  - *Frontend* → `fe-add-phi-form`, `fe-add-clinical-view`, `fe-add-search-filter`, `fe-add-break-glass-modal`, `fe-add-auth-ui`, `fe-add-realtime-alerts`, `fe-add-abdm-flow`, `fe-add-audit-inspector`.
  - *Database* → `add-migration`, `db-add-query-optimization`.
  - *Testing/QA* → `test-*` and `write-invariant-test`, chosen to cover each canonical scenario.
- **Pick by clinical standard in play:** US Core FHIR → `*-fhir-*` skills; LOINC labs → `be-add-lab-integration`; RxNorm/allergies → `be-add-medication-service`; HL7 v2.x → `be-add-hl7-processor`.
- **Human-only override:** if a selected skill touches Auth, RBAC/data projections, billing, migrations on real data, the PHI allow-list, or clinical-data storage decisions (`AGENTS.md` §2), the agent may *prepare* the change but must **never merge it** — it routes to a named engineer.

The traceability matrix is the source of truth that keeps skill choice deterministic and auditable rather than a guess.

---

## Worked Example: "Add insurance-verified appointment booking"

This mirrors the real case study recorded in [`docs/journal.md`](docs/journal.md) (`HOLD-2026-10-07-01`).

**Raw request:**
> *"When the appointment is created, also support insurance verification code. If the patient has Medicaid or Medicare, allow front desk to see the clinical reason so they can determine if pre-authorization is required."*

**Step 1 — Groom, don't code.** `feature-grooming` drafts the spec against `06_appointment_scheduling_booking.md` and the 4 canonical scenarios.

**Step 2 — Decompose & flag.** Two `[CLIENT-DECISION-NEEDED]` issues surface immediately:
- `insuranceVerificationCode` and payer enums (Medicaid/Medicare) are **not in the spec** → Stop Condition 1 (Spec Silence).
- "Let front desk see the clinical reason" **projects clinical data to a non-clinical role** → Stop Condition 6 (Clinical Data Leakage), a Human-Only area (§2).

**Step 3 — HOLD & report (do not guess).** The agent halts, writes a structured `HOLD` to `docs/journal.md`, and stops. It invents nothing.

```
[HOLD - Stop Condition Triggered]
Reason 1: Projecting clinical reason to 'front_desk' violates HIPAA Minimum Necessary + AGENTS.md Human-Only policy.
Reason 2: 'insuranceVerificationCode' / payer eligibility are absent from spec 06.
Required Human Actions: specify a discrete non-clinical field, update spec.
Status: HOLD
```

**Step 4 — Human decides.** A named engineer records `RESOLVED-HOLD`: never expose raw clinical reasons; add a discrete non-clinical `requiresPreAuth: boolean` to the appointment DTO and amend the spec.

**Step 5 — Resume via skill selection.** With the resolution in hand: `be-add-scheduling` + `add-endpoint` implement the redacted booking route, `fe-add-search-filter` surfaces the front-desk view, and `write-invariant-test` adds a planted-fault test asserting the front-desk projection returns `requiresPreAuth` and *never* clinical text (access test hits the API as `fd_mark_miller`, expects 403 on clinical fields + an audit entry).

**Step 6 — Prove & verify.** `npm run verify` is green; reverting the redaction fails the invariant test; the audit event fires on read/write; the traceability row is updated.

**Result:** The front desk gets exactly the signal it needs — a non-clinical "pre-auth required" flag — while clinical notes stay masked, every access is audited, and the risky decision remains owned by a human.

---

## Limitations

- **Reference kit, not a running app.** This repository currently ships rules, specs, skills, and governance docs; the executable service and full harness live outside it (see ADR-003 — QA invariant harness is a separate track).
- **Human-in-the-loop is mandatory, not optional.** Any risky change stalls on sign-off; throughput is intentionally capped by review and stop conditions.
- **Synthetic data only.** No real patient data may flow through agents, fixtures, or logs — so behaviors involving production-shape data edge cases must be validated separately.
- **Skill coverage is bounded by the traceability matrix.** New clinical standards or domains not yet mapped to a skill require human-authored spec + skill work before agents can safely act.
- **Model-fragility.** Skills and `AGENTS.md` must be re-run on a sample task whenever the underlying model or a key library changes (`guideLine.md` §16); guarantees are not automatically portable across models.
- **Not a legal HIPAA determination.** The PHI guardrails encode best practices, not a substitute for a Compliance Officer's review or a signed BAA where required.

---

## License

Released under the [MIT License](LICENSE). Copyright (c) 2026 Pinaki Batabyal.
