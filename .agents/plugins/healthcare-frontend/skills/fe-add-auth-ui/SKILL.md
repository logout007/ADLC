---
name: fe-add-auth-ui
description: Standardized procedure to build secure healthcare authentication screens with MFA input, inactivity countdown timers, and automatic 15-minute session lock.
---

# Skill: Frontend - Healthcare Authentication & Session Lock

Follow this procedure whenever developing user login, MFA challenges, or clinical session management interfaces.

## HIPAA Technical Safeguards
1. **Emergency Session Timeout:** Inactivity must trigger a session lock after 15 minutes for clinical workstations (§164.312(a)(2)(iii)).
2. **Two-Factor Authentication (MFA):** Healthcare provider accounts require mandatory second-factor verification (SMS/Authenticator app).
3. **No Credential Storage:** Password fields must strictly use `type="password"`, disable browser caching on shared terminals, and clear state on unmount.

## Implementation Steps

### 1. Build Login Form Component
- Form elements: Email / Provider NPI, Password, Show/Hide password toggle.
- On valid initial credentials, trigger MFA challenge step.

### 2. Implement MFA Challenge Step
- 6-digit numeric input with auto-focus and auto-advance.
- "Resend Code" button with 60-second cooldown timer.

### 3. Handle Shared-Phone Identity Disambiguation
- When multiple family members share a mobile number, the verification endpoint returns `needsProfileSelection: true` and a short-lived `selectionToken` (no session token yet).
- Render a "Who's signing in?" picker displaying registered profiles with masked DOBs (`**/**/YYYY`) and relationship labels (`PARENT`, `GUARDIAN`, `SPOUSE`, `CHILD`, `SELF`).
- On profile selection, exchange `selectionToken` + `identityId` for the final authenticated `sessionToken`.

### 4. Implement Inactivity Session Lock Hook (`useClinicalInactivityLock`)
- Track user events (`mousemove`, `keydown`, `touchstart`).
- At 13 minutes of inactivity: Show modal warning: *"Your clinical session will expire in 2 minutes due to inactivity. Click to remain signed in."*
- At 15 minutes of inactivity: Clear tokens from memory, wipe displayed clinical state, and redirect to session lock screen requiring re-authentication.

### 5. Profile Switching Context (`ProfileContext`)
- Maintain active `identityId` in persistent context.
- Support switching between registered identities on the same phone via dedicated switch endpoint/action without re-entering credentials.

### 6. Stop Conditions
- **HOLD:** If the spec proposes removing or extending the 15-minute inactivity timeout for clinical accounts.
- **HOLD:** If the application attempts to issue a full session token when `needsProfileSelection` is true without explicit user selection.
