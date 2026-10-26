---
name: test-api-fhir-conformance
description: Standardized procedure to test RESTful APIs and US Core FHIR R4 conformance, pagination, and error responses (Skill 4.5).
---

# Skill: API & FHIR Conformance Testing

Follow this procedure to test HTTP endpoints and FHIR R4 schema compliance.

## Pre-conditions
- Target endpoints and FHIR resource profiles (US Core v3.1.1+) must be documented in `docs/spec/`.

## Procedure
1. **HTTP Contract Conformance:**
   - Verify standard status codes:
     - 200 OK for successful queries
     - 201 Created with Location header for new resources
     - 400 Bad Request with descriptive JSON error for schema violations
     - 403 Forbidden for insufficient permissions
     - 404 Not Found for missing entities
     - 409 Conflict for double-booking or concurrency collisions
2. **FHIR Schema Validation:**
   - Validate response structures against FHIR R4 schemas (`resourceType`, `id`, `meta`, `identifier`).
   - Validate terminology bindings (e.g. SNOMED, LOINC, RxNorm code systems).
   - Test FHIR `OperationOutcome` error formatting.
3. **Query Parameters & Pagination:**
   - Test offset/limit or cursor-based pagination.
   - Test FHIR search parameters (e.g. `_sort`, `patient`, `date`).
