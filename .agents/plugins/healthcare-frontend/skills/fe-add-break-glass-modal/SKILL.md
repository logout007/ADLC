---
name: fe-add-break-glass-modal
description: Standardized procedure to implement emergency "Break-Glass" override UI with mandatory justification capture, compliance warnings, and immediate audit dispatch.
---

# Skill: Frontend - Add Break-Glass Modal

Follow this procedure whenever implementing emergency clinical access override UI components.

## Clinical Safety & Audit Standards
- **IHE Basic Audit Log Profile (BALP):** Break-glass activation must dispatch an immediate high-priority audit event to compliance sinks.
- **Strict Time-Boxing:** Access granted under break-glass is temporary and must automatically expire after **4 hours**.
- **Accountability & Mandatory Friction:** Prevents accidental or routine misuse by requiring explicit category selection, minimum explanation length, and legal attestation.

## Implementation Steps

### 1. Modal Trigger & Role Authorization
- Render `BreakGlassModal` only when an authenticated clinician (`role === 'clinician'`) attempts to view an unassigned patient record and receives HTTP 403 `UNASSIGNED_CARE_TEAM_RESTRICTION`.
- Block non-clinicians (front-desk, billing) from ever seeing or invoking the break-glass interface.

### 2. Form Requirements & State Validation
- **Emergency Category Dropdown:**
  - `CODE_BLUE_EMERGENCY`: Acute cardiac or respiratory arrest.
  - `TRAUMA_ADMISSION`: Unconscious emergency department patient.
  - `UNSCHEDULED_CROSS_COVERAGE`: Urgent physician cross-coverage during colleague absence.
- **Detailed Clinical Justification Textarea:**
  - Mandatory minimum length: **20 characters**.
  - Real-time character counter showing progress.
- **Legal Attestation Checkbox:**
  - *"I certify under penalty of institutional disciplinary action that this emergency access is required for immediate patient care."*
- **Submit Button State:**
  - Strictly disabled until a reason is selected, text length $\ge 20$, and attestation checkbox is checked.

### 3. API Dispatch & Active Session Warning Banner
- Submit payload to `POST /api/access/break-glass`.
- On success, mount a persistent top viewport warning banner:
  ```html
  <div class="emergency-override-banner" role="alert">
    <IconAlertTriangle /> EMERGENCY OVERRIDE ACTIVE (Expires in: 03:59:45) - Access is logged and audited.
  </div>
  ```

### 4. Automatic Expiration & Session Masking
- Attach countdown timer: at 4 hours, automatically wipe displayed clinical state from memory and return to locked screen.

### 5. Stop Conditions
- **HOLD:** If the spec permits Break-Glass access without capturing mandatory justification text.
