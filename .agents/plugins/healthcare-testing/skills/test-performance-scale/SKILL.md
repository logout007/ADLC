---
name: test-performance-scale
description: Standardized procedure to test performance, concurrency, response time thresholds, and database query efficiency under clinical load (Skill 4.6).
---

# Skill: Performance & Concurrency Testing

Follow this procedure to test performance targets and prevent concurrency bottlenecks in healthcare systems.

## Pre-conditions
- Performance budgets must be established (e.g. API responses $< 200$ms p95, Stat alerts $< 30$s).

## Procedure
1. **Response Time Benchmarking:**
   - Execute baseline timing tests against clinical search and appointment slot availability endpoints.
   - Assert p95 response times remain strictly within the SLA threshold.
2. **Concurrency & Race Condition Testing:**
   - Simulate simultaneous double-booking requests for the exact same slot by two concurrent clients.
   - Verify optimistic locking or transactional isolation ensures exactly one booking succeeds (201) and the second receives 409 Conflict.
3. **Database Query Optimization:**
   - Check query execution plans to verify indexed lookups on `mrn`, `patientId`, and `appointmentDate`.
   - Assert zero N+1 query patterns when loading appointment lists with provider metadata.
