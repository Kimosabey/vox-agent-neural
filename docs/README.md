# VoxAgent Neural - Documentation Base

Welcome to the **VoxAgent Neural** documentation. This project provides a production-grade implementation of a real-time speech transcription system using an Agentic Hybrid architecture.

**Author**: Harshan Aiyappa — Senior Hybrid Engineer

---

## 📚 Documentation Index

### Architecture & Design

- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Comprehensive architecture analysis
  - Current Agentic Hybrid approach (LiveKit + WebSocket)
  - Alternative patterns comparison (Standard SFU vs Direct WebSocket)
  - Performance metrics & optimization
  - Cost analysis for Enterprise scale
  - Recommendation for low-latency neural inference

---

## 🚀 Quick Start

### Prerequisites
- Python 3.11+
- Node.js 18+
- FFmpeg installed

### Backend Setup (Inference Engine)
```bash
cd backend
python -m venv venv311
venv311\Scripts\activate  # Windows
pip install -r requirements.txt
python main.py dev
```

### Frontend Setup (Client UI)
```bash
cd frontend
npm install
npm run dev
```

### Environment Variables
Create `.env` file in root:
```env
LIVEKIT_URL=wss://your-livekit-url
LIVEKIT_API_KEY=your-key
LIVEKIT_API_SECRET=your-secret
```

---

## ✨ Features

![User Journey](./images/user_flow.png)

- 🎙️ **Real-time Agentic Transcription** using `faster-whisper` neural agents.
- ⚡ **Optimized Inference** (2-5s latency on CPU-only hardware).
- 🔄 **Persistent Agent Connection** for instant-on recording cycles.
- ✅ **Premium UI** (Modern monochrome theme, Framer Motion animations).
- ✅ **Telemetry Tracking** (Turnaround time (TAT) and latency monitoring).
- ✅ **Streaming Visualizer** (High-fidelity 60fps audio amplitude monitoring).
- ✅ **Scribe Management** (Export, copy, and download transcripts).

---

## 🏗️ Project Structure

```
vox-agent-neural/
├── frontend/               # React + TS application
│   ├── src/
│   │   ├── components/    # Reusable UI components
│   │   ├── hooks/         # useLiveKit & useRecorder hooks
│   │   ├── pages/         # Page components
│   │   └── lib/           # Logic utilities
│
├── backend/               # FastAPI + Neural Scribe Engine
│   ├── main.py           # Application entry & WS handler
│   ├── requirements.txt  # Python dependency manifest
│
├── docs/                 # Product Documentation
│   ├── README.md         # This file
│   └── ARCHITECTURE.md   # System analysis
│
└── .env                  # Environment configuration
```

---

## 🔐 Security & Reliability

### Production Best Practices
- [x] Environment variables for secret management.
- [x] CORS configuration.
- [x] JWT token-based authentication for LiveKit rooms.
- [ ] Implement WSS (WebSocket Secure) for encrypted egress.
- [ ] Add rate limiting at the API Gateway level.
- [ ] Implementation of circuit-breakers for fallback inference engines.

---

## 📝 API Reference (Control Plane)

### WebSocket Node
**URL**: `ws://localhost:8000/ws`

**Binary/Data Payload:**
```json
{
  "type": "audio_chunk",
  "data": "base64_encoded_audio",
  "timestamp": 0
}
```

**Agent Response:**
```json
{
  "type": "transcript",
  "text": "transcribed telemetry",
  "turnaround_ms": 2450,
  "isFinal": true
}
```

---

## 📄 License

MIT License - see [LICENSE](LICENSE) for details

---

**Last Updated**: January 15, 2026  
**Author**: Harshan Aiyappa — Senior Hybrid Engineer
