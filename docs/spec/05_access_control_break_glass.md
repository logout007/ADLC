# Domain Specification 05: Access Control & Break-Glass Protocols

## 1. Domain Scope & Clinical Context
Governs Role-Based Access Control (RBAC), Attribute-Based Access Control (ABAC), patient consent scopes, field-level Minimum Necessary projections, and the emergency "Break-Glass" protocol for clinicians treating unassigned patients in acute distress.

---

## 2. Healthcare Role & Data Projection Matrix

| Role | Demographics & Scheduling | Clinical Notes & SOAP | Lab Values & Vitals | Prescriptions & Allergies | Billing & Insurance |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`patient`** | Full (Own data) | View (Signed only) | View (Resulted only) | View (Active only) | View (Own statements) |
| **`clinician` (Attending)**| Full | Full Read/Write | Full Read/Order | Full Read/Write | Read (Coverage status) |
| **`clinician` (Unassigned)**| Masked | **Blocked** (Break-Glass needed) | **Blocked** | **Blocked** | Blocked |
| **`nurse` / Care Team**| Full | View & Triage notes | View & Record vitals | View & Administer | Blocked |
| **`front_desk`** | Full (Schedule/Name) | **STRICTLY BLOCKED** | Status only (No values)| Status only | View (Eligibility/Copay)|
| **`billing`** | View | ICD-10 / CPT only | Blocked | Blocked | Full Read/Write |
| **`auditor`** | Masked / Anonymized | Audit logs only | Audit logs only | Audit logs only | Audit logs only |

---

## 3. The Emergency "Break-Glass" Protocol
In acute medical emergencies (e.g. unconscious trauma patient, emergency cross-coverage), a licensed clinician who is **NOT** the assigned attending physician must be able to temporarily bypass access blocks to view life-critical allergies and history.

### Break-Glass Mandatory Rules:
1. **Explicit Justification:** The clinician must provide an emergency reason (e.g. `ACUTE_RESUSCITATION`, `CODE_BLUE_CROSS_COVERAGE`, `TRAUMA_ADMISSION`) with $\ge 20$ characters of clinical explanation.
2. **Time-Limited Access:** Elevated access expires automatically after **4 hours**.
3. **Heightened Audit & Notification (IHE BALP):**
   - Emits an immediate high-priority audit event: `EMERGENCY_BREAK_GLASS_TRIGGERED`.
   - Sends real-time notification to the Hospital Privacy Officer and the patient's primary attending physician.
   - Enters the retrospective clinical review queue for compliance verification within 24 hours.

---

## 4. Personas & Scenarios (Given / When / Then)

### Scenario 1: Emergency Break-Glass Execution (Happy Emergency Path)
- **Given** clinician `dr_unassigned_guest` faces an acute clinical emergency with `p_sarah_connor`
- **When** the clinician submits `POST /api/access/break-glass` with justification reason `ACUTE_RESUSCITATION` and notes `"Patient unconscious in trauma bay, checking life-threatening medication allergies"`
- **Then** the system grants a 4-hour temporary elevated clinical projection token
- **And** writes an immutable audit record with actor NPI, patient MRN, and emergency justification
- **And** routes an alert to the Hospital Privacy Officer.

### Scenario 2: Automatic Expiration After 4 Hours (Edge Case)
- **Given** an active Break-Glass override session granted at 10:00 AM
- **When** the clinician attempts to query patient records at 2:01 PM (past 4-hour limit)
- **Then** the system invalidates the override token
- **And** returns HTTP 403 Forbidden with error code `BREAK_GLASS_EXPIRED`
- **And** prompts the clinician to re-authenticate or re-justify if care is still ongoing.

### Scenario 3: Missing Required Clinical Justification Rejects Activation (Failure Path)
- **Given** clinician `dr_unassigned_guest` initiates a Break-Glass override
- **When** the clinician clicks submit with an empty justification or fewer than 20 characters
- **Then** the API rejects the request with HTTP 422 Unprocessable Entity and error code `JUSTIFICATION_TOO_SHORT`
- **And** no temporary emergency access is granted.

### Scenario 4: Non-Clinician Attempts Break-Glass (Misuse Path)
- **Given** front-desk staff `fd_mark_miller` attempts to invoke the break-glass endpoint
- **When** the security layer checks credentials
- **Then** the request is rejected immediately with HTTP 403 Forbidden and code `CLINICIAN_CREDENTIALS_REQUIRED`
- **And** an alert `UNAUTHORIZED_PRIVILEGE_ESCALATION_ATTEMPT` is recorded in the security audit log.
