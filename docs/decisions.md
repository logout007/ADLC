# Architecture & Design Decisions Log

## ADR-001: Role-Based Field Projections via Strict DTO Whitelisting
- **Status:** Accepted
- **Context:** Front-desk staff need appointment details (patient name, clinician, time slot) but must be blocked from seeing clinical notes and diagnoses under HIPAA Minimum Necessary rule.
- **Decision:** Implement an outbound response projection layer that explicitly white-lists allowed fields based on caller role (`PROJECT_ROLE_DTO`), rather than blacklisting sensitive fields.
- **Alternatives Rejected:**
  - *Blacklist filtering (`delete obj.clinicalNotes`):* Rejected because if new clinical fields (e.g. `prescription`, `icd10`) are added later, a blacklist would silently leak them by default.
  - *Separate database tables and dual queries:* Rejected for this phase as premature schema complexity; projection at API serialization boundary provides uniform security.
- **Reversibility:** Low risk. Can be swapped for an automated GraphQL / Prisma schema projection without breaking REST contract.

## ADR-002: Anti-Corruption Layer (ACL) for External EHR/FHIR Integrations
- **Status:** Accepted
- **Context:** Connecting internal clinical systems to external hospital EHR systems (Epic, Cerner) risks internal PHI leakage and schema coupling.
- **Decision:** Mandate the `add-adapter` skill where all third-party interactions use contract fixtures during development and explicit bi-directional transformation DTOs.
- **Alternatives Rejected:**
  - *Direct domain model pass-through to EHR APIs:* High risk of accidental field leakage and breaking changes if vendor schema updates.
- **Reversibility:** High modularity. Adapters can be modified independently of core domain models.

## ADR-003: Lifecycle Boundary: Developer Implementation vs QA Invariant Harness
- **Status:** Accepted
- **Context:** The team requested a clean separation of roles: developers focus on domain implementation, repeatable skills, and specs; QA owns the invariant test harness and planted-fault verifications.
- **Decision:** Repository developer workspace focuses on:
  1. Detailed healthcare specifications in `docs/spec/`.
  2. Repeatable development skills in `.agents/skills/` (`add-endpoint`, `add-adapter`, `add-notification`, `add-migration`).
  3. Pre-commit PHI safeguards and command-blocking guardrails.
  QA team owns the invariant test suites, planted-fault injection runner, and compliance regression gates in a dedicated harness repository / pipeline.
