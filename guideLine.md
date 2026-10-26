
AI-DLC Standard
Building healthcare software with AI agents: the AI writes, machinery enforces, and people decide where it matters.
Principles
Five rules apply to every AI-assisted change, on every project.

1. Spec first. Every task starts from a written requirement with a testable acceptance criterion, never from a loose prompt.
2. Rules advise, machinery enforces. Anything that matters has a test, a hook or a CI gate behind it, not just an instruction.
3. Prove the checks work. Every critical test is shown failing on a planted fault before we trust it.
4. Stop, don't guess. When the spec is silent or contradicts itself, the agent stops and reports. It never invents a value, a rule or a field.
5. People own the risky parts. The AI writes code; a named engineer owns architecture, review and correctness, and some areas are never merged by an agent.
   The lifecycle
   Every project runs these ten layers. Layers marked today we already practise; the rest are the standard we are moving to.

Layer
What the standard requires
Tooling
1
Intent
Product goal, a spec, and a backlog where every item has testable acceptance criteria. One item in progress at a time. Today: product goal
Spec in repo, build journal, issue tracker
2
Instructions
A short AGENTS.md, per-module rules for risky areas, skills for repeatable procedures, pinned versions. Today: skills
AGENTS.md, CLAUDE.md pointer, .agents/skills/, Context7 or equivalent for current docs
3
Guardrails
Hooks block dangerous commands before they run; no secrets or production access; allow-listed network; the agent cannot edit harness files
Claude Code hooks and sandbox, protected harness folder
4
Proof
Compliance tests written first; one verify command locally and in CI; contract tests against recorded fixtures; E2E on synthetic personas
npm run verify, Vitest/Jest, Playwright, GitHub Actions
5
Proving the checks
Each critical test has a planted fault it must catch and a correct twin it must pass
Planted-fault fixtures, mutation testing
6
Review
Fresh-context AI reviewer (security, PHI, spec fidelity), then human review. Human-only areas never agent-merged. Today: PR gates
Reviewer agent, CODEOWNERS, required reviews
7
Stop rules
Three-attempt rule; written stop conditions; HOLD and wait; no unproven success claims; every run lists what was not checked
Rules file, stop-time checks
8
Healthcare data
Synthetic data only; PHI tagged and scrubbed; no PHI to AI tools without a BAA; separate environments
Fixture factories, PHI lint in CI, log scrubber
9
Traceability
Spec clause to test mapping; decision log with rejected options; corrections log; PRs linked to milestones
Docs in repo, conventional commits
10
Release and learning
Protected main, required checks, staging smoke test, human-approved production release with rollback; every correction becomes a rule or test
GitHub rulesets, staging environment, retro
Definition of done
An AI-written change is done only when every line below is true. If the agent cannot meet a line, it stops and reports; it never lowers the bar.

1. The verify command passes with no skipped or disabled tests.
2. A test covers the change and fails if the change is reverted.
3. No new enum value, error code or config key appears that is not in the spec.
4. No PHI field reaches a log, fixture, snapshot or unapproved destination.
5. Every new access to patient data writes an audit event, shown by a test.
6. The PR links its requirement or milestone and lists anything not checked.
7. The fresh-context AI reviewer has passed it, and a named engineer has approved it.
8. If it touches a human-only area, it is marked for human review and not merged by the agent.
   Human-only areas and stop conditions
   Agents fail hardest where a mistake becomes a legal or safety event, so these areas are proposed by the agent and merged only by a person.
   Human-only areas (never agent-merged)
   • Authentication, sessions and MFA
   • Role-based access and data projections (who sees which fields)
   • Payments, billing and capture logic
   • Database migrations on any environment with real data
   • The PHI allow-list and any vendor that receives PHI
   • Any decision about what clinical data is stored or shown
   Stop conditions (the agent writes HOLD, reports and waits)
   Condition
   What the agent does
   The spec is silent on a value, name or rule
   Reports the gap; does not invent one
   Two parts of the spec contradict each other
   Reports both; does not choose
   A compliance or invariant test would need changing to pass
   Stops; the change is wrong, not the test
   Three attempts at the same fix have failed
   Stops trying variations and writes a structured report
   A third-party API behaves differently from its fixture
   Reports the difference as a spec variance
   A task would show clinical data to a non-clinical role
   Stops and asks
   Healthcare data rules for AI tools
   No real patient data ever reaches an AI tool or a non-production environment. Everything else follows from that.
   • Synthetic only. Dev and staging use fixture factories and named synthetic personas. A CI check fails the build on real-looking data: real email domains, valid card numbers, ID-number patterns.
   • No copying from production. If a bug can't be reproduced with synthetic data, raise it; never copy the record.
   • PHI is tagged in the code. Tagged fields are scrubbed from logs and error reports, and blocked from any destination without a signed BAA.
   • AI vendors. Never paste client data, credentials or PHI into an AI tool. Any AI service that processes PHI in a product must be under a BAA and approved by the project lead.
   • Least access for agents. Agents get no production credentials, no client secrets and only the network hosts the task needs.
   • Client code stays in client repos. Agents work only in the client's organisation; nothing is copied to personal or external repos.
   Personas and scenarios
   Every project defines its user personas and scenarios during business requirements, and the same set drives the tests. A requirement without a persona and a scenario is not ready to build.
   During requirements
   • List every persona who touches the system, including staff and automated actors: for example patient, caregiver or family member, clinician, front-desk or liaison staff, billing, admin, and SYSTEM jobs.
   • For each persona, write what they may see and do, and what they must never see. This becomes the access matrix.
   • Write each requirement as a scenario in given/when/then form, naming the persona.
   • Cover four kinds of scenario for every feature: happy path, edge cases, failures (third party down, payment declined, late lab result), and misuse (wrong role, crafted request, duplicate submit).
   • Include special cases that are common in healthcare: patients in more than one programme, minors or dependants where relevant, out-of-area addresses, accessibility needs, and patients who return after cancelling.
   During testing
   • Each persona has a named synthetic seed record (for example p_new, p_eligible, p_past_due), so the agent and QA use the same data.
   • Each scenario maps to at least one automated test; the spec-to-test mapping shows any scenario with no test.
   • Access tests run per persona against the API, not the UI: every role tries to read fields it must not see and must get a 403 and an audit entry.
   • End-to-end journeys run for every persona from first contact to the end state, on both happy and failure paths.
   • QA owns the persona and scenario list, and reviews it at each milestone; new scenarios found in testing or production are added to the list and to the tests.
   Scenario type
   Example
   Test level
   Happy path
   Patient completes intake and books a consult
   E2E
   Edge case
   Patient enrolled in two programmes with different eligibility
   Integration + E2E
   Failure
   Lab webhook never arrives; polling fallback picks up the result
   Contract + integration
   Misuse
   Non-clinical staff requests lab values directly from the API
   API access test
   Compliance
   Payment capture attempted before eligibility is recorded
   Invariant test
   Repository starter kit
   Every new project starts with these files on day one, before feature code.
   File or folder
   Purpose
   AGENTS.md
   Short, stable rules: vocabulary, hard rules, human-only areas, stop conditions
   CLAUDE.md
   One line: "See AGENTS.md"
   .agents/skills/
   Repeatable procedures: add-endpoint, add-migration, add-adapter, add-notification, write-invariant-test
   docs/spec/
   The requirements, personas and scenarios
   docs/traceability.md
   Spec clause or scenario to test mapping
   docs/decisions.md
   Decision log: what was chosen, what was rejected, how to reverse it
   docs/journal.md
   Build journal: one item in progress, HOLDs and corrections
   test/invariants/
   Compliance tests, each with a planted fault and a correct twin
   test/fixtures/
   Synthetic factories and named persona seed data
   .claude/ hooks and settings
   Command-blocking hooks, sandbox and network allow-list
   .github/
   Required checks, CODEOWNERS for human-only areas, PR template with "not checked" section
   package.json scripts
   setup, lint, typecheck, test, test:invariants, test:contracts, test:e2e, verify
   Project setup and governance
   The layers above cover how code is built. These items decide whether a project succeeds from day one; items 1–4 and 7 must be in place before the first feature PR.
   Before any code is written
9. Spec review and gap list. The team reads the full spec, writes a numbered gap list (contradictions, missing rules, unclear workflows) and gets the client's answers in writing before building.
10. Architecture and threat model. A data-flow diagram showing where PHI goes, the main threats and how each is mitigated, and the key architecture decisions recorded in docs/decisions.md. Reviewed by the project lead.
11. Day-zero setup checklist. Repo access in the client's organisation, cloud accounts, BAA confirmed, secrets store, three environments (dev, staging, production), and the AI tools approved for this client.
12. Roles. Named owners for: delivery and client communication, the harness and skills, QA with personas and scenarios, and review of human-only areas.
    Role
    Owns
    Project lead
    Architecture, human-only reviews, client sign-offs
    Harness owner
    AGENTS.md, skills, hooks, CI gates
    QA lead
    Personas, scenarios, traceability, E2E and access tests
    Engineers
    Features through the definition of done
    During the build
13. Security testing in CI. Dependency vulnerability scan, secret scanning and static analysis on every PR; high and critical findings block merge.
14. Non-functional testing. Accessibility (WCAG 2.1 AA), page-load and API response targets, and a basic load test, run before each milestone demo.
15. Client cadence and scope control. A weekly written update (shipped, in progress, blocked, risks), a live demo per milestone against its acceptance criteria, written client sign-off, and a change-request process for anything outside the agreed scope.
16. Testing the skills. Skills and AGENTS.md are versioned like code. When the model or a key library changes, each skill is re-run on a sample task and its output checked.
    Running and improving
17. Operations readiness. Monitoring and alerts, an incident runbook, backups with a tested restore, and a rollback plan, all in place before go-live.
18. Metrics. Track per project: PRs passing CI first time, review rejections, defects found after staging, invariant failures caught, and hours against estimate. Review them at each milestone.
19. AI tool governance. Approved models and tools per client, an open-source licence check on dependencies and generated code, and an AI usage cost budget.
