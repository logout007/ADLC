---
name: fe-add-clinical-components
description: Standardized procedure to build reusable clinical React UI components (Patient Summary Card, Medication List, Lab Results Table with abnormal indicators).
---

# Skill: Frontend - Reusable Clinical Component Library

Follow this procedure whenever developing reusable UI components for electronic health records, patient summaries, or clinical charts.

## Design & Usability Standards
1. **Clinical Context Prominence:** Critical clinical flags (Allergies, Resuscitation Status / DNR, Fall Risk) must be immediately visible at the top of the chart.
2. **Abnormal Indicators:** Highlight abnormal lab values with red/amber badges, but always accompany color with a distinct icon and screen-reader text (`aria-label`).
3. **Skeleton Loading:** Always render skeleton loading states while clinical data is in-flight to prevent layout shifts.

## Implementation Steps

### 1. Build Patient Summary Card
- Header component rendering:
  - Synthetic Name, Age / DOB, Gender Identity, MRN.
  - Allergy pills (Red pill for severe anaphylaxis, Amber for mild).
  - Quick action buttons: Book Appointment, Message Clinician, Break-Glass (if unassigned).

### 2. Build Medication List Component
- Table rendering active vs past medications:
  - Columns: Medication Name & Dose, Route & Frequency, Prescriber, Refills Remaining, Status Badge.
  - Highlight potential interactions with a warning icon and tooltip explaining the interaction.

### 3. Build Lab Results Component
- Table displaying recent observations:
- Columns: Test Name (LOINC), Result Value & UCUM Unit, Reference Range, Status (`FINAL`/`PENDING`), Flag.
- Include sparkline or trending icon for historical comparison.

### 4. Build PWA Platform Installation Banner
- Detect platform and browser environment:
  - **Android / Chromium:** Intercept `beforeinstallprompt` event and trigger browser install dialog on button click.
  - **iOS Safari:** Display step-by-step instructions (*"Tap Share, then Add to Home Screen"*).
- Implement 14-day dismissal suppression in browser storage.
- Auto-hide when running in standalone display mode (`display-mode: standalone`).

### 5. Build Care Activity Composite Score Ring
- Visual circular progress indicator (0% to 100%) tracking patient engagement and records completeness:
  1. Upcoming care set (at least 1 future reminder scheduled).
  2. Medications tracked (active medications list present).
  3. Recent clinical activity (encounter or record within last 90 days).
  4. Attention queue clear (no failed or pending document reviews).
- **Mandatory Subtitle:** Explicitly label as a record completeness metric (e.g., *"3/4 up to date — not a clinical or diagnostic score"*).

### 6. Accessibility & Mobile Responsiveness
- Ensure all interactive elements have touch targets $\ge 44 \times 44\text{px}$.
- Full keyboard Tab navigation support.

### 7. Stop Conditions
- **HOLD:** If component designs omit reference ranges or units of measurement for lab results.
- **HOLD:** If the Care Activity Score is presented as a medical or diagnostic risk score rather than an administrative completeness indicator.
