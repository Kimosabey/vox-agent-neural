# 🏗️ System Architecture

## 1. High-Level Design (HLD)

VoxAgent Neural operates as a **Hybrid Control Plane** that orchestrates real-time audio (UDP) and reliable text data (TCP). It decouples the "Ear" (Frontend) from the "Brain" (Backend) using an Agentic Pattern.

```mermaid
graph TD
    User([👤 User / Microphone]) -->|RTP Audio (UDP)| LiveKit[☁️ LiveKit SFU]
    
    subgraph "The Neural Agent"
        Agent[🐍 Python Controller] -->|Subscribe Track| LiveKit
        Agent -->|1. VAD Check| CPU[Logic Gate]
        CPU -->|2. Quantized Inference| Model[Faster-Whisper (INT8)]
        Model -->|3. Text Event| WS[WebSocket Server]
    end
    
    WS -->|Transcript (TCP)| User
```

### Core Components
1.  **Frontend (React Agent)**: Manages the WebRTC connection state and renders the "Stealth UI" (a minimalist, distraction-free interface).
2.  **LiveKit Cloud (SFU)**: Handles NAT traversal, bitrate adaptation, and packet loss concealment.
3.  **Neural Engine (FastAPI)**: A Python service running the `faster-whisper` inference loop and a WebSocket broadcaster.
4.  **Quantized Model**: An optimized CTranslate2 version of OpenAI's Whisper `base` model, capable of running on non-GPU instances.

---

## 2. Low-Level Design (LLD)

### The Inference Pipeline
To achieve sub-second latency without a GPU:
1.  **Ingestion**: Audio arrives as Opus frames via LiveKit `AudioStream`.
2.  **Transcoding**: `FFmpeg` converts Opus (48kHz) to PCM (16kHz Mono) in RAM.
3.  **Buffering**: A customized `RollingWindow` collects 3 seconds of audio.
4.  **Inference**:
    *   **Beam Size**: 1 (Greedy decoding for speed).
    *   **Temperature**: 0 (Deterministic).
    *   **Compute**: INT8 (Vectorized CPU instructions).
5.  **Egress**: Text JSON pushed to WebSocket clients.

### Data Schema (Transcription Event)
```json
{
  "type": "transcription",
  "id": "uuid-v4",
  "text": "Hello world.",
  "is_final": true,
  "confidence": 0.98,
  "metrics": {
    "audio_duration": 2.45,
    "inference_time": 0.32
  }
}
```

---

## 3. Decision Log

| Decision | Alternative | Reason for Choice |
| :--- | :--- | :--- |
| **Hybrid (LiveKit + WS)** | Pure WebSockets | **Quality**. Sending raw audio over WebSocket (TCP) causes "Head-of-Line Blocking" on poor networks. LiveKit (UDP) allows audio to skip dropped packets, maintaining real-time flow. |
| **Faster-Whisper** | OpenAI Cloud API | **Latency & Privacy**. Cloud APIs add ~500ms network RTT and expose user data. Local inference allows for "Interim Results" (seeing the text appear as you speak). |
| **Process Separation** | Monolith | **Scale**. We run the Inference Engine as a separate process. If the AI crashes (OOM), the WebSocket connection remains alive to tell the user "System Rebooting". |

---

## 4. Key Patterns

### The "Turnaround Time" (TAT) Metric
We treat Lag as a first-class citizen.
*   Every audio chunk is stamped with `t0` (Client Time).
*   The text result includes `t1` (Processing Time).
*   The UI calculates `Delta = t1 - t0`.
*   **Feedback**: If Delta > 2000ms, the UI glows Orange to warn the user: "You are speaking faster than I can think."

### Agentic State Sync
The Frontend doesn't just "show text". It mirrors the Agent's state.
`Connecting` -> `Handshaking` -> `Listening` -> `Thinking` -> `Cooldown`.
This prevents the "Is it working?" confusion common in voice apps.
