---
name: test-integration-systems
description: Standardized procedure to test integrations between healthcare components, EHR adapters, and databases (Skill 4.2).
---

# Skill: Integration Testing Healthcare Systems

Follow this procedure to test interactions between services, database persistence, and external EHR adapters.

## Pre-conditions
- External EHR/LIS endpoints must use recorded mock fixtures (Anti-Corruption Layer).
- Real production credentials or network connections are strictly forbidden.

## Procedure
1. **Mock Setup & Fixture Isolation:**
   - Load synthetic contract fixtures from `test/fixtures/`.
   - Ensure external HTTP requests are intercepted by test doubles.
2. **Transaction & Database Testing:**
   - Test end-to-end data flow: Controller $\rightarrow$ Service $\rightarrow$ Repository.
   - Assert database rollback behavior on transaction failures.
3. **Workflow Journey Scenarios:**
   - Execute full lifecycle: Patient intake $\rightarrow$ Consent validation $\rightarrow$ Appointment booking $\rightarrow$ EHR sync.
4. **Resilience Testing:**
   - Test downstream timeout handling and retry policies with exponential backoff.
5. **Audit Verification:**
   - Confirm every integration transaction records an audit log entry.
