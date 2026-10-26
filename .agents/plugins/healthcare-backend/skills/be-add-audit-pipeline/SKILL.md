---
name: be-add-audit-pipeline
description: Standardized procedure to implement immutable HIPAA audit trail logging with tamper-evident hashing and zero PHI in telemetry.
---

# Skill: Backend - Add Audit Pipeline

Follow this procedure whenever implementing or attaching audit trail logging to clinical, demographic, or financial interactions.

## Regulatory Standards
- **HIPAA Technical Safeguards:** 45 CFR § 164.312(b) (Audit Controls).
- **ONC 2015 Edition Cures Act:** **§ 170.315(d)(2)** (Auditable Events and Tamper-Resistance) & **§ 170.315(d)(3)** (Audit Reports).
- **Standard Specification:** **ASTM E2147-18** (Standard Specification for Audit and Disclosure Logs for Use in Health Information Systems).
- **Clock Synchronization:** **RFC 5905** Network Time Protocol (NTP) synchronized UTC timestamps.

## Implementation Steps

### 1. Define ASTM E2147-18 Audit Event Schema
Every audit event record MUST capture:
- `id`: Globally unique event ID.
- `timestamp`: UTC ISO-8601 timestamp synchronized via NTP.
- `actorId`: User ID, clinician NPI, or worker service account.
- `actorRole`: `patient` | `clinician` | `nurse` | `front_desk` | `billing` | `system`.
- `action`: Standardized verb (`CREATE`, `READ`, `UPDATE`, `DELETE`, `EXPORT`, `BREAK_GLASS`).
- `resourceType`: Target entity (`Patient`, `Encounter`, `DocumentReference`, `Observation`, `Prescription`).
- `resourceId`: Entity unique identifier.
- `outcome`: `SUCCESS` | `DENIED` | `FAILED`.
- `correlationId`: Distributed tracing header (`X-Correlation-ID`).
- `prevHash`: Cryptographic SHA-256 hash of the previous audit record (tamper-evident blockchain/chaining).

### 2. Tamper-Resistance & Detection Engine (ONC § 170.315(d)(2))
- Compute cryptographic hash for every event:
  $$\text{hash} = \text{SHA-256}(\text{prevHash} + \text{timestamp} + \text{actorId} + \text{action} + \text{resourceId} + \text{outcome})$$
- Store logs in **Write-Once-Read-Many (WORM)** append-only tables or dedicated audit log sinks.
- Provide verification script `verifyAuditIntegrity()` that traverses the chain and detects if any record was inserted, modified, or deleted.

### 3. Strict PHI Redaction from Telemetry
- Free-text clinical notes, diagnoses, and medical narratives MUST NOT be stored in audit logs.
- Only log structural identifiers, metadata keys, and status codes.

### 4. Audit Report Generation (ONC § 170.315(d)(3))
- Provide sortable querying by patient ID, provider NPI, date range, and action type for compliance review and breach investigations.

### 5. Stop Conditions
- **HOLD:** If any feature request attempts to allow disabling audit logs or bypass logging for administrative roles.
