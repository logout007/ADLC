---
name: fe-add-abdm-flow
description: Standardized procedure to implement National Health ID (ABHA/ABDM) enrollment, Aadhaar dual-OTP verification, holder comparison reconciliation, and time-bound consent management.
---

# Skill: Frontend - National Health ID (ABHA / ABDM) Workflow

Follow this procedure whenever implementing national health identity creation, verification, holder reconciliation, or consent management interfaces.

## Privacy & Regulatory Safeguards
1. **Ephemeral National ID Numbers:** Aadhaar or other government ID numbers must strictly remain ephemeral in memory during the transaction. Never log, persist in localStorage/cookies, or display full ID numbers.
2. **Dual-OTP Communication Check:** When the communication phone differs from the Aadhaar-registered phone, ensure a separate secondary OTP verification step completes before final confirmation.
3. **Time-Bound Consent & Revocation:** Patient data sharing consents must have explicit validity dates (e.g., 365 days) and provide an immediate revocation mechanism.
4. **Holder Discrepancy Reconciliation:** When government identity data (name, DOB, gender) differs from the clinical profile, present an explicit comparison confirmation dialog before linking records.

## Implementation Steps

### 1. Government ID Enrolment Wizard
- **Method Selection:** Support `GOV_ID_OTP` (Aadhaar), `MOBILE_OTP_DOCUMENT`, or biometric verification where supported.
- **Explicit Consent:** Display versioned consent wording with an active confirmation checkbox (`consentAccepted: true`, `consentVersion`).
- **OTP Verification:** 6-digit auto-focused numeric input with resend cooldown timer.

### 2. Handle Dual-OTP Phone Check (`abhaEnrolCheckMobile`)
- After initial Aadhaar OTP verification, call the mobile check endpoint.
- If `mobileOtpRequired` is true, trigger a second verification screen to validate the communication mobile number before confirming enrollment.

### 3. Holder Mismatch Comparison Component (`AbhaHolderCompare`)
- When ABDM demographic data differs from the portal profile:
  - Display a structured comparison table showing Name, Year of Birth, and Gender for both the profile and the ABHA record.
  - Highlight mismatches with clear badges.
  - Require the user to explicitly confirm that both records belong to the same individual (`confirmSamePerson: true`).

### 4. ABHA Card & QR Display
- Render ABHA card and QR assets received as Base64 payloads from the backend.
- Provide PNG / PDF download triggers without exposing raw ABDM session tokens to client storage.

### 5. Time-Bound Consent Management
- Display hospital sharing authorizations with countdown badges:
  - Show active days remaining, expiration warning when within 30 days.
  - Provide 1-tap **Renew** (extends term) and **Revoke** actions.
  - Render granular Health Information Type (HI-Type) toggles: `OPConsultation`, `Prescription`, `DiagnosticReport`.

### 6. Stop Conditions
- **HOLD:** If the specification proposes caching Aadhaar numbers in client-side persistent storage.
- **HOLD:** If government ID holder mismatches are automatically overwritten without explicit patient confirmation.
