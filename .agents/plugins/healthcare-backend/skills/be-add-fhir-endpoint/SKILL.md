---
name: be-add-fhir-endpoint
description: Standardized procedure to implement a FHIR R4 compliant RESTful API endpoint with US Core profiling, search filtering, and field-level RBAC projection.
---

# Skill: Backend - Add FHIR R4 Endpoint

Follow this procedure whenever implementing or exposing healthcare resources via FHIR R4 REST APIs (e.g., `Patient`, `Encounter`, `DocumentReference`, `Observation`, `Condition`).

## Market Standards & Regulatory Conformance
- **HL7 FHIR Release 4 (R4):** Conforms to official **US Core Implementation Guide v3.1.1 / v4.0.0** (required by ONC 2015 Edition Cures Act).
- **US Core Required Clinical Notes (LOINC):**
  - Consultation Note: `11488-4`
  - Discharge Summary: `18842-5`
  - History & Physical (H&P) Note: `34117-2`
  - Progress Note: `11506-3`
  - Emergency Department Note: `34111-5`
- **Audit Logging:** Emits ASTM E2147-18 compliant audit event via FHIR `AuditEvent`.

## Implementation Steps

### 1. Spec & Resource Profile Check
- Verify resource schema in `docs/spec/`.
- Ensure mandatory elements (cardinality `1..*`) and "Must Support" flags are mapped from domain models.

### 2. Route & Middleware Setup
- Set up route: `GET /fhir/R4/:resourceType` and `GET /fhir/R4/:resourceType/:id`.
- Attach `authMiddleware` to extract caller role and SMART-on-FHIR scopes (`patient/Patient.read`, `user/DocumentReference.read`).
- Apply `auditMiddleware` capturing `X-Correlation-ID` and query parameters.

### 3. Implement Search & Filtering
- Support standard US Core search parameters:
  - `DocumentReference?patient=[id]&category=clinical-note`
  - `DocumentReference?patient=[id]&type=http://loinc.org|11506-3`
  - `Patient?_id=[id]` or `Patient?identifier=[mrn]`
  - `Observation?patient=[id]&category=vital-signs&date=ge[date]`
- Implement pagination with `_count` and `_offset` returning a FHIR `Bundle` with `type: "searchset"`.

### 4. Field Projection (Minimum Necessary Rule)
- For non-clinical roles (e.g. `front_desk`, `billing`), strip clinical extensions, narrative text, and attachment binaries.
- Only return authorized demographic metadata matching the role's access level.

### 5. Error Responses (FHIR OperationOutcome)
- Standardized error format matching US Core:
  ```json
  {
    "resourceType": "OperationOutcome",
    "issue": [
      {
        "severity": "error",
        "code": "forbidden",
        "diagnostics": "Role 'front_desk' is restricted from clinical notes under HIPAA Minimum Necessary rule",
        "expression": ["DocumentReference.content"]
      }
    ]
  }
  ```

### 6. Stop Conditions
- **HOLD:** If the spec lacks mapping for local EHR fields to official US Core FHIR profiles.
