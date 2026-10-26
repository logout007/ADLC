---
name: db-add-query-optimization
description: Standardized procedure to optimize healthcare database queries, index clinical identifiers (MRN, DOB, NPI), and avoid N+1 query patterns.
---

# Skill: Database - Clinical Query Optimization & Indexing

Follow this procedure whenever adding new query patterns, scaling clinical repositories, or diagnosing slow search responses.

## Principles
1. **Index High-Frequency Clinical Keys:** MRN, EMPI, Provider NPI, and Encounter Timestamps must be indexed to ensure sub-500ms lookups.
2. **Prevent N+1 in Medical Charts:** When loading patient charts, use joins or batch aggregation pipelines to retrieve active medications, problems, and vitals in single queries.
3. **Compound Indexes for Date Ranges:** Composite indexes (e.g. `{ patientId: 1, encounterDate: -1 }`) are essential for longitudinal clinical timelines.

## Implementation Steps

### 1. Identify Query Patterns in `docs/spec/`
- Determine if the query is point-lookup (by MRN), range-lookup (vitals in last 7 days), or text search (patient name prefix).

### 2. Implement Index Definitions
- In schema definition:
  ```javascript
  // Patient lookups
  PatientSchema.index({ mrn: 1 }, { unique: true });
  PatientSchema.index({ 'legalName.family': 1, 'legalName.given': 1, birthDate: 1 });

  // Encounter timelines
  EncounterSchema.index({ patientId: 1, encounterDate: -1 });

  // Lab observation trending
  ObservationSchema.index({ patientId: 1, loincCode: 1, resultTimestamp: -1 });
  ```

### 3. Use Lean Projections
- Always project only required fields (`select: 'id mrn legalName birthDate'`) rather than fetching entire patient histories.

### 4. Query Plan Verification
- Run `EXPLAIN` on queries to verify index utilization (`COLLSCAN` is prohibited for clinical search routes).

### 5. Stop Conditions
- **HOLD:** If a query proposal requires a table scan across the entire historical patient database.
