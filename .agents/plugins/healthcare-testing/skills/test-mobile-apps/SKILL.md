---
name: test-mobile-apps
description: Standardized procedure to test mobile patient portals and clinician mobile rounding apps on device boundaries, offline sync, and security locks (Skill 4.8).
---

# Skill: Mobile Health App Testing

Follow this procedure when testing mobile healthcare applications (React Native / iOS / Android).

## Pre-conditions
- Target mobile flows must specify offline requirements and session inactivity parameters.

## Procedure
1. **Inactivity Session Lockout:**
   - Verify app automatically locks behind biometric or PIN screen after 15 minutes of inactivity or upon entering background mode.
2. **Offline Data Storage Security:**
   - Verify unencrypted PHI is NEVER written to SQLite, Realm, or Async Storage.
   - Verify offline data queues use SQLCipher with hardware keystore backed encryption.
3. **Network Resilience & Sync:**
   - Test offline form capture $\rightarrow$ network reconnection $\rightarrow$ idempotent queue drain.
   - Assert conflict resolution when clinical records are updated simultaneously on server and mobile.
4. **Device Permissions & Hardware Protection:**
   - Verify camera and microphone access are requested only in active telehealth sessions.
   - Verify screen capture / screenshot blocking on sensitive clinical screens (FLAG_SECURE on Android).
