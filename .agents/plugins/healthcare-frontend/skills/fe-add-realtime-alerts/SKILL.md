---
name: fe-add-realtime-alerts
description: Standardized procedure to implement real-time WebSocket connections for stat clinical alerts, abnormal lab notifications, and badge updates.
---

# Skill: Frontend - Real-Time Clinical Alerts

Follow this procedure whenever implementing live notifications for critical lab values, emergency room triage alerts, or care team chats.

## Clinical Safety Standards
1. **Audible & Visual Cues for Stat Values:** Critical values (e.g. cardiac enzyme spike, severe hypoxemia) require both an audio tone and an interruptive visual badge.
2. **Reconnection & Queuing:** If the WebSocket connection drops, automatically reconnect with exponential backoff and fetch missed events via REST sync.
3. **No Unencrypted PHI in Payloads:** Alert notification payloads must only contain alert type, urgency level, and patient identifier; full clinical details are fetched via authenticated API call.

## Implementation Steps

### 1. Establish Secure WebSocket Connection
- Connect to `wss://...` using authenticated session token.
- Subscribe to specific channels (e.g. `provider-alerts:${providerId}`).

### 2. Handle Inbound Alert Payloads
- On message received:
  - If priority = `CRITICAL`: Render persistent toast notification with action button ("Review Lab Result") and play alert tone.
  - If priority = `ROUTINE`: Increment unread notifications badge counter.

### 3. Connection State UI
- Render subtle status indicator in header:
  - Green dot: Live connected.
  - Amber dot: Reconnecting.
  - Red dot: Offline (prompt user to check internet).

### 4. Stop Conditions
- **HOLD:** If alert payload specifications include plain-text patient names or diagnoses over unauthenticated channels.
