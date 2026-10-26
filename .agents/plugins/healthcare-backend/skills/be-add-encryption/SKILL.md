---
name: be-add-encryption
description: Standardized procedure to implement field-level data encryption (AES-256-GCM), key management, and cryptographic protection for sensitive healthcare fields.
---

# Skill: Backend - Field-Level Data Encryption

Follow this procedure whenever storing or processing highly sensitive fields (SSN, financial details, psychotherapy notes).

## Principles
1. **Encrypt Before Persist:** Sensitive fields must be encrypted at the application layer prior to database insertion.
2. **Authenticated Encryption:** Use AES-256-GCM to ensure both confidentiality and tamper-resistance.
3. **Zero Hardcoded Keys:** Master encryption keys must be injected via secure KMS / environment variables; never checked into git.

## Implementation Steps

### 1. Identify Sensitive Fields
- Check field classifications in `docs/spec/`.
- Target fields: SSN, payment tokens, sensitive behavioral health notes.

### 2. Implement Cryptographic Utility
- Create `src/utils/crypto.ts`:
  - `encryptField(plaintext: string): { ciphertext: string, iv: string, authTag: string }`
  - `decryptField(ciphertext: string, iv: string, authTag: string): string`
- Use 12-byte cryptographically secure random IV for each encryption.

### 3. Implement Schema Pre-Save & Post-Find Hooks
- In Mongoose/Sequelize models:
  - Pre-save hook: Encrypt sensitive attributes.
  - Post-find hook: Decrypt only when authorized by caller's role.

### 4. Key Rotation Support
- Include key version identifier in encrypted payload metadata:
  - `{ v: 1, iv: "...", tag: "...", data: "..." }`
- Support on-demand re-encryption under new key version.

### 5. Stop Conditions
- **HOLD:** If the spec requests persisting raw unencrypted SSN or banking credentials.
