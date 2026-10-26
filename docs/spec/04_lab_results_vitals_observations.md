# Domain Specification 04: Lab Results, Vital Signs & Observations

## 1. Domain Scope & Clinical Context
Governs laboratory test orders, results, abnormal/critical value notification cascades, biometric vital signs tracking (Blood Pressure, Heart Rate, Respiratory Rate, SpO2, Temperature), and longitudinal clinical trending.

---

## 2. Terminology & Coding Standards
- **Lab Observations:** LOINC (Logical Observation Identifiers Names and Codes) (e.g. `14682-9` for Serum Creatinine, `2093-3` for Total Cholesterol, `718-7` for Hemoglobin).
- **Vital Signs Standards:** HL7 FHIR `Observation` resource complying with the **US Core Vital Signs Profile (v3.1.1)**.
- **Units of Measure:** Unified Code for Units of Measure (UCUM) (e.g., `mg/dL`, `mm[Hg]`, `%`, `cel`, `mmol/L`).
- **Critical Threshold Standards:** Medical laboratory critical value reporting guidelines (CAP / CLIA standards).

---

## 3. Data Schema & Critical Value Thresholds

| Observation Element | Code (LOINC) | Standard Normal Range | Critical Abnormal Threshold (Stat Alert) |
| :--- | :--- | :--- | :--- |
| **Systolic BP** | `8480-6` | `90 - 120 mm[Hg]` | `< 70` or `> 180 mm[Hg]` (Hypertensive Crisis / Shock) |
| **Diastolic BP** | `8462-4` | `60 - 80 mm[Hg]` | `< 40` or `> 110 mm[Hg]` |
| **Heart Rate** | `8867-4` | `60 - 100 beats/min` | `< 40` or `> 140 beats/min` (Severe Bradycardia / Tachycardia) |
| **Oxygen Saturation (SpO2)**| `2708-6` | `95 - 100 %` | `< 88 %` (Severe Hypoxemia) |
| **Serum Potassium (K+)** | `2823-3` | `3.5 - 5.0 mmol/L` | `< 2.8` or `> 6.2 mmol/L` (Cardiac Arrhythmia Risk) |
| **Blood Glucose (Fast)** | `1558-6` | `70 - 99 mg/dL` | `< 50` (Hypoglycemia) or `> 400 mg/dL` (DKA Risk) |

---

## 4. Clinical Safety Invariants
- **INV-LAB-001 (Critical Value Escalation):** Whenever a lab or vital sign result exceeds a Critical Abnormal Threshold, the system MUST emit an immediate high-priority alert to the attending clinician within 30 seconds.
- **INV-LAB-002 (Immutable Lab Results):** Once resulted by an external LIS (Laboratory Information System), a lab observation cannot be overwritten. Updates must be posted as an amended observation with status `amended` or `corrected`.
- **INV-LAB-003 (Age & Demographic Reference Ranges):** Reference ranges displayed in both patient and clinician views must be dynamically contextualized by patient age and gestational/sex status.

---

## 5. Personas & Scenarios (Given / When / Then)

### Scenario 1: Vital Signs Recorded During Intake (Happy Path)
- **Given** nursing staff enters vitals for patient `p_sarah_connor`: BP 118/76, HR 72, SpO2 99%
- **When** the vitals are saved via `POST /api/observations/vitals`
- **Then** the observations are validated against UCUM units, mapped to normal baseline, and stored with status `FINAL`
- **And** the longitudinal vitals chart updates immediately.

### Scenario 2: Age-Adjusted Pediatric Heart Rate Evaluation (Edge Case)
- **Given** synthetic infant patient `p_baby_oliver` (age 6 months)
- **When** a heart rate of 130 beats/min is entered
- **Then** the clinical logic evaluates age-specific pediatric ranges (normal infant HR is 100-160)
- **And** correctly classifies the observation as `NORMAL` rather than flagging tachycardia.

### Scenario 3: Critical Potassium Value Triggering Stat Escalation (Critical Failure / Alert Path)
- **Given** an incoming LIS HL7 ORU result for `p_sarah_connor` reports Serum Potassium `6.4 mmol/L`
- **When** the adapter processes the LOINC `2823-3` observation
- **Then** the system flags interpretation as `CRITICAL_HIGH`
- **And** triggers an immediate urgent notification to attending clinician `dr_elena_vance`
- **And** writes an immutable audit record: `CRITICAL_LAB_VALUE_ALERT_EMITTED`.

### Scenario 4: Front-Desk Query for Lab Status (Misuse / Projection Check)
- **Given** a patient has pending or completed biopsy results
- **When** front-desk staff `fd_mark_miller` queries the lab order status via `GET /api/lab-orders/:id`
- **Then** the API returns order status metadata (`PENDING` or `COMPLETED`)
- **And** the actual lab test result values, pathology text, and clinical interpretations are stripped from the response.
