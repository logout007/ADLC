---
name: add-migration
description: Standardized procedure to draft database migrations for healthcare schemas with synthetic seed verification and rollback scripts.
---

# Skill: Add Healthcare Schema Migration

Follow this procedure whenever proposing a database schema modification or data transformation.

## Human-Only Warning
**Per `AGENTS.md` Section 2:** Database migrations running on persistent/production data stores are **Human-Only** and must **NEVER** be merged by an AI agent. The AI agent may propose the migration script and rollback script, but a named senior engineer must review and execute it.

## Implementation Steps

### 1. Spec Alignment
- Confirm the new entity, column, or relationship is explicitly documented in `docs/spec/`.
- Ensure no new enum values or config keys are invented.

### 2. Backward Compatibility
- Follow expand-and-contract pattern:
  - Step 1: Add new column as nullable or with safe default.
  - Step 2: Update application code to read/write both versions.
  - Step 3: Contract deprecated columns only after verification.
- **Never perform destructive `DROP COLUMN` or `TRUNCATE` operations in feature migrations.**

### 3. Migration and Rollback Pair
- Every migration `YYYYMMDDHHMMSS_name.up.sql` MUST have an accompanying `YYYYMMDDHHMMSS_name.down.sql` rollback script.
- Verify migration idempotency (can run repeatedly without corrupting data).

### 4. Synthetic Seed Updates
- Update synthetic personas and seed factories to reflect the schema update.
- Ensure zero real patient identifiers are introduced in seed files.

### 5. PR Tagging
- Mark PR as touching a **Human-Only Area: Database Migrations**.
- Assign lead engineer via `CODEOWNERS`.
