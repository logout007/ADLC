---
name: be-add-hl7-processor
description: Standardized procedure to parse, validate, and generate HL7 v2.x healthcare messages (ADT, ORM, ORU) with ACK/NAK generation and MLLP handling.
---

# Skill: Backend - Parse & Generate HL7 v2.x Messages

Follow this procedure whenever integrating with hospital systems, LIS, or pharmacy feeds communicating via HL7 v2.x standards.

## Principles
1. **Strict Segment Validation:** MSH, PID, and clinical segments (OBR, OBX, ORC) must conform to standard field delimiter rules (`|`, `^`, `~`, `\`, `&`).
2. **Immediate Acknowledgement:** Every inbound message must generate an immediate `ACK` (success) or `NAK` (error) with matching message control ID.
3. **Synthetic Isolation:** All HL7 test feeds must use synthetic patient identifiers (`p_synthetic_*`).
4. **Idempotency on MessageControlID:** HL7 engines re-transmit unacknowledged feeds. Duplicate `MessageControlID` entries must be acknowledged immediately without re-mutating patient charts or orders.

## Implementation Steps

### 1. Spec & Message Type Alignment
- Confirm the HL7 message type in `docs/spec/`:
  - `ADT^A01` / `ADT^A08`: Admission, transfer, or demographic update.
  - `ORM^O01`: Lab or imaging order.
  - `ORU^R01`: Unsolicited observation results (lab/radiology).

### 2. Implement Inbound Message Parser
- Parse message headers (`MSH` segment):
  - Extract `SendingApplication`, `SendingFacility`, `MessageControlID`, and `HL7Version`.
- Parse patient identification (`PID` segment):
  - Extract Patient ID, synthetic name components (`PID-5`), and DOB (`PID-7`).
- Parse clinical segments:
  - For `ORU^R01`: Iterate over `OBX` segments to extract observation identifier (LOINC), value, units (UCUM), reference range, and abnormal flag (`H`, `L`, `CRIT`).

### 3. Implement Acknowledgement Generator
- Generate matching `MSA` segment:
  - Success: `MSA|AA|<MessageControlID>`
  - Error: `MSA|AE|<MessageControlID>|<ErrorMessage>`

### 4. Transform to Internal Domain Model
- Map parsed HL7 data into internal domain entities via an Anti-Corruption Layer (ACL).
- Never allow raw pipe-delimited strings to leak into frontend responses.

### 5. Audit Logging
- Emit audit record: `action: HL7_MESSAGE_PROCESSED`, `messageControlId`, `messageType`, `outcome`.

### 6. Stop Conditions
- **HOLD:** If the inbound HL7 feed contains custom Z-segments not documented in `docs/spec/`.
