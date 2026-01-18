# 🎤 Interview Cheat Sheet: VoxAgent Neural

## 1. The Elevator Pitch (2 Minutes)

"VoxAgent Neural is a **Real-Time Agentic Transcription Platform**.
It allows users to have low-latency voice interactions with AI models *without* relying on expensive cloud GPUs.
I achieved this by building a **Hybrid Architecture**:
1.  **LiveKit** handles the WebRTC signaling and connectivity (solving the 'Head-of-Line Blocking' problem of generic WebSockets).
2.  **Faster-Whisper** runs locally on the CPU (using CTranslate2 quantization) to deliver 'good enough' transcription in <500ms.
It utilizes an 'Agentic' pattern where the UI and the Brain are decoupled but synchronized via a Control Plane."

---

## 2. "Explain Like I'm 5" (The Translator)

"Imagine you are giving a speech at the UN.
*   **Normal STT**: Is like recording speech, mailing the tape to a translator, and getting a letter back next week.
*   **VoxAgent**: Is like a simultaneous translator sitting in the booth.
*   **The Trick**: My translator (The AI) listens to small chunks (3 seconds) and writes them down immediately. If he gets too busy, he raises his hand (Backpressure) so you slow down, rather than him simply quitting."

---

## 3. Tough Technical Questions

### Q: Why use Faster-Whisper instead of OpenAI API?
**A:** "Data Sovereignty and Latency.
*   **Privacy**: Audio never leaves the server. This is critical for medical/legal apps.
*   **Streaming**: OpenAI API is request/response. I needed a continuous stream. Faster-Whisper allows me to maintain a 'Context Buffer' so the AI remembers the start of the sentence while decoding the end."

### Q: Why combine LiveKit AND WebSockets? Isn't that redundant?
**A:** "It's a strategic separation of concerns.
*   **LiveKit (UDP)**: Is perfect for Audio Transport (RTP) because it handles packet loss gracefully (Audio glitch > Latency).
*   **WebSockets (TCP)**: Is perfect for the Transcript Text. We need reliability here—dropping a word is worse than a 50ms delay.
So providing a 'Hybrid' channel gives the best of both protocols."

### Q: How do you handle CPU spikes on the Edge?
**A:** "I implemented **Quantization (INT8)**.
Standard Whisper uses 32-bit floats. By compressing the model weights to 8-bit integers, I reduced memory usage by 4x and increased speed by 3x on consumer CPUs, with less than 1% accuracy loss."
