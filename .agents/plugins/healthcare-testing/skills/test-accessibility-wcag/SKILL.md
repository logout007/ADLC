---
name: test-accessibility-wcag
description: Standardized procedure to test clinical user interfaces for WCAG 2.1 AA accessibility and assistive technology compatibility (Skill 4.7).
---

# Skill: Accessibility Testing (WCAG 2.1 AA)

Follow this procedure to ensure healthcare web applications are usable by clinicians and patients with visual, motor, or cognitive disabilities.

## Pre-conditions
- UI components must target WCAG 2.1 AA compliance.

## Procedure
1. **Automated Axe-Core Auditing:**
   - Run automated axe-core scans across all rendered views (Intake form, Appointment calendar, Patient card).
   - Assert 0 critical or serious accessibility violations.
2. **Keyboard Navigation & Focus Management:**
   - Verify all interactive controls (buttons, modals, dropdowns) are focusable and operable via Keyboard (`Tab`, `Enter`, `Escape`).
   - Verify focus trapping inside modals (e.g. Break-Glass modal, booking confirmation).
3. **Color Contrast & Medical Status Indicators:**
   - Assert contrast ratio $\ge 4.5:1$ for normal text, $\ge 3:1$ for large text.
   - Assert abnormal/critical lab indicators do NOT rely solely on color (must include text label or distinct icon like `CRITICAL [!]`).
4. **Screen Reader Semantic Markup:**
   - Verify appropriate `aria-live` announcements for Stat clinical alerts.
   - Verify explicit `aria-label` or `<label>` tags for all clinical input fields.
