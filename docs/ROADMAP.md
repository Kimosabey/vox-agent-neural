# 🚀 Future Roadmap: Neural Agent Evolution

This document outlines the planned architectural evolution for **VoxAgent Neural**. While the current implementation focuses on low-latency WebSocket streams for single-user sessions, the roadmap points towards a fully agentic, WebRTC-native ecosystem.

## 🏗️ Architectural Evolution

The transition moves from a **Direct-Stream Handler** to an **Autonomous LiveKit Worker**.

| Module              | Current (WebSocket/Hybrid)                     | Future (Native WebRTC Agent)                   |
| :------------------ | :--------------------------------------------- | :--------------------------------------------- |
| **Audio Transport** | Client Mic -> WebSocket -> Backend             | Client Mic -> LiveKit (WebRTC) -> Backend      |
| **Backend Role**    | HTTP/WebSocket Server                          | LiveKit Worker (Subscribes to Track)           |
| **Frontend**        | Publish Mic + Stream MediaRecorder via WS      | Publish Mic Track, Receive DataChannel         |
| **Processing**      | Direct Neural Inference on incoming bytes       | Noise Gate -> VAD -> Buffer -> Neural Ingress   |
| **Output**          | WebSocket Message                              | LiveKit DataChannel (Encrypted)               |

---

## 🛠️ Planned Enhancements

### Phase 1: Neural Pipeline Optimization
- **Voice Activity Detection (VAD)**: Integration of `webrtcvad` to filter background silence before neural inference.
- **Audio Pre-processing**: Implementation of noise calibration and gating to improve transcription accuracy in high-decibel environments.
- **Dynamic Buffering**: Transitioning from fixed 3s chunks to responsive buffer windows (2.4s - 4.5s) based on speech cadence.

### Phase 2: Native Agent Integration
- **LiveKit Agent Framework**: Converting the backend into a dedicated `Participant` that subscribes to audio tracks natively.
- **Data Channel Egress**: Moving away from standard WebSockets to LiveKit's ultra-low-latency Data Channels for real-time text delivery.
- **Diarization**: Support for multi-speaker identification during collaborative sessions.

---

## ✅ Expected Resulting Architecture

**Voice Capture** → **WebRTC Relay** → **Python Neural Worker** → **VAD/Inference Pipeline** → **Encrypted Token Stream** → **Agentic UI**.

---

**Last Updated:** January 15, 2026  
**Author:** Harshan Aiyappa — Senior Hybrid Engineer
