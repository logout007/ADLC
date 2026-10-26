---
name: fe-add-clinical-view
description: Standardized procedure to build accessible clinical dashboard components (vital signs charts, lab trending, problem lists) complying with WCAG 2.1 AA and medical display standards.
---

# Skill: Frontend - Add Clinical View Component

Follow this procedure whenever developing UI components that present clinical data to patients or medical staff.

## Healthcare UI Standards
1. **WCAG 2.1 Level AA Accessibility:** Minimum contrast ratio 4.5:1 for normal text, keyboard navigability, full screen-reader support (ARIA labels).
2. **Clinical Safety Colors:** Never rely solely on color to convey abnormal values. Pair colors with distinct icons/shapes (e.g. Red + triangle warning icon for critical values, Green + check for normal).
3. **Contextual Units:** Always display UCUM units of measurement alongside clinical values (e.g., `120 mmHg`, `4.2 mmol/L`).
4. **PII Masking & Privacy Screen:** Support toggleable privacy mask (e.g., concealing patient name/MRN when displaying in shared clinic environments).
5. **Zero PHI in URL Parameters:** Routing and queries must NEVER contain patient names, DOBs, or diagnoses in URLs (e.g. use `/patients/:uuid/vitals`, never `/patients?name=Sarah`).
6. **Telemetry Sanitization:** Frontend error handlers (Sentry/Datadog) must redact all component state before dispatching crash reports.

## Implementation Steps

### 1. Component Props & Sensitivity Tagging
- Define TypeScript interface with explicit clinical fields.
- Include role prop (`userRole`) to conditionally render clinical notes vs stripped front-desk view.

### 2. State & Data Presentation
- Display reference ranges alongside observation values:
  ```html
  <span class="lab-value critical-high" aria-label="Potassium 6.2 mmol/L, Critical High, Normal range 3.5 to 5.0">
    <IconAlertTriangle aria-hidden="true" /> 6.2 mmol/L (Normal: 3.5 - 5.0)
  </span>
  ```
- Render empty states clearly (e.g. "No active allergies documented" rather than a blank box).

### 3. Print-Friendly Styling
- Include `@media print` CSS rules for clinical charts:
  - Remove navigation menus, headers, and background colors.
  - Ensure high-contrast black/white readability for paper medical records.

### 4. Client-Side Security & Inactivity Timeout
- Attach inactivity detection hook (`useSessionTimeout`).
- Automatically mask screen after 15 minutes of inactivity in clinical views.

### 5. Stop Conditions
- **HOLD:** If design mocks show clinical notes accessible to non-clinical roles.
