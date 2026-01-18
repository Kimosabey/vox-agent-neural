# 🛡️ Failure Scenarios & Resilience

> "Real-time AI is high-bandwidth and CPU-intensive. Things break."

This document details how VoxAgent Neural handles resource saturation and network disconnects.

## 1. Failure Matrix

| Component | Failure Mode | Impact | Recovery Strategy |
| :--- | :--- | :--- | :--- |
| **Neural Engine** | CPU Saturation (100%) | **High**. Latency spikes to >5s. | **Backpressure**. The WebSocket server drops audio chunks if the processing queue depth > 3. Client shows "System Overload" toast. |
| **WebRTC** | ICE Connection Fail | **Critical**. No Audio. | **Signaling Retry**. The LiveKit SDK automatically attempts to switch TURN servers if P2P fails. |
| **WebSocket** | Disconnect | **Major**. No Transcripts. | **Auto-Reconnect**. The `useLiveKit` hook acts as a resilient wrapper, attempting to re-dial the WS connection every 2s for 5 attempts. |

---

## 2. Deep Dive: Circuit Breaking

### The Problem
During long monologues, the `faster-whisper` model can get stuck trying to decode infinite audio without a pause token, causing CPU usage to lock at 100%.

### The Solution: VAD-Triggered Reset
We use the WebRTC VAD (Voice Activity Detection) metadata.
*   **Trigger**: If `speech_active` is True for > 30 seconds straight.
*   **Action**: Force-flush the Whisper buffer.
*   **Result**: The user might lose 0.5s of audio, but the system un-freezes.

---

## 3. Resilience Testing

### Test 1: Network Interrupt
1.  Start talking.
2.  Toggle WiFi Off/On.
3.  **Expectation**: 
    *   UI turns **Red** (Disconnected).
    *   UI turns **Yellow** (Reconnecting).
    *   UI turns **Green** (Connected).
    *   Session history is preserved.

### Test 2: The "Rap God" Test (Load)
1.  Play a fast-paced podcast at 2x speed into the mic.
2.  **Expectation**: The "Turnaround Time" (TAT) metric will climb.
3.  **Fail Condition**: If TAT > 5s, the system should start dropping chunks to catch up.
