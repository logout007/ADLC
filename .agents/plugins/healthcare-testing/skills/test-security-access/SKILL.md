---
name: test-security-access
description: Standardized procedure to test authentication, RBAC, ABAC, and privilege escalation vulnerabilities (Skill 4.4).
---

# Skill: Security & Access Control Testing

Follow this procedure to test role-based access control, privilege boundaries, and authentication security.

## Pre-conditions
- The access matrix in `docs/spec/` must explicitly state permitted and forbidden operations for each role.

## Procedure
1. **Persona Access Matrix Testing:**
   - Execute test matrices for all roles (`patient`, `front_desk`, `clinician`, `billing`, `admin`).
   - For every endpoint, verify permitted roles receive 200/201 and prohibited roles receive 403 Forbidden.
2. **Horizontal Privilege Escalation (IDOR):**
   - Verify patient `p_sarah_connor` cannot access records belonging to another patient ID.
3. **Emergency Break-Glass Testing:**
   - Verify unassigned clinician cannot access emergency chart without active Break-Glass session.
   - Verify Break-Glass session strictly expires after 4 hours.
4. **Security Vulnerability Checks:**
   - Test for parameter pollution, injection attacks, and expired/tampered JWT tokens.
5. **Audit Assertion:**
   - Verify all 403 authorization failures emit an audit security event (`ACCESS_DENIED`).
