---
name: fe-add-search-filter
description: Standardized procedure to build patient search and clinical filtering interfaces with autocomplete, fuzzy matching, MRN lookup, and active filter tags.
---

# Skill: Frontend - Patient Search & Filter Interface

Follow this procedure whenever implementing search bars, patient lookup dialogs, or filter sidebars.

## Privacy & Security Standards
1. **Debounced Search:** Debounce search queries by $\ge 300\text{ms}$ to prevent excessive backend load and audit log spam.
2. **Masked Suggestions:** Autocomplete dropdown suggestions must mask sensitive details (e.g. show `Sarah C. (DOB: **/**/1984, MRN: ***-1234)`).
3. **Audit Event Trigger:** Patient searches trigger an audit query log entry on the backend.

## Implementation Steps

### 1. Build Search Input
- Text input supporting search by Name, MRN, or DOB.
- Provide clear visual indicators when searching is active (spinner).
- Include "Clear Search" button to reset state.

### 2. Implement Autocomplete Dropdown
- Render keyboard-navigable list (Up/Down arrow keys, Enter to select).
- Handle empty state gracefully ("No matching synthetic patients found").

### 3. Implement Multi-Criteria Filters
- Filter sidebar / chips:
  - Encounter Type: Outpatient, Telehealth, Inpatient.
  - Date Range: Last 30 days, Last 6 months, Custom date picker.
  - Provider: Dropdown of care team clinicians.
  - Status: Active, Completed, Cancelled.

### 4. Accessibility
- Use ARIA combobox pattern (`role="combobox"`, `aria-autocomplete="list"`).
- Announce result count to screen readers via `aria-live="polite"`.

### 5. Stop Conditions
- **HOLD:** If the spec permits unrestricted wildcard searches that return entire patient databases without pagination.
