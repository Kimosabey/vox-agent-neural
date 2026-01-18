# 🚀 Getting Started with VoxAgent Neural

> **Prerequisites**
> *   **Python 3.11** (Required for `faster-whisper`)
> *   **Node.js 18+**
> *   **LiveKit Cloud Account** (Free Tier)

## 1. Environment Setup

### A. LiveKit Credentials
1.  Sign up at [LiveKit Cloud](https://cloud.livekit.io).
2.  Create a new project.
3.  Copy your `API Key` and `Secret Key`.

### B. Configuration (`.env`)
Create a `.env` file in the root:
```ini
LIVEKIT_URL=wss://your-project.livekit.cloud
LIVEKIT_API_KEY=API...
LIVEKIT_API_SECRET=Secret...
```

---

## 2. Installation & Launch

### Step 1: Start Neural Engine (Backend)
The backend requires a Python environment for the AI model.

```bash
cd backend
python -m venv venv
# Windows
.\venv\Scripts\activate
# Mac/Linux
source venv/bin/activate

pip install -r requirements.txt
python main.py
```
*Wait for "Neural Engine Ready" log. This loads the 500MB Whisper model into RAM.*

### Step 2: Start Control Plane (Frontend)
```bash
cd frontend
npm install
npm run dev
```

---

## 3. Usage Guide

1.  **Open Browser**: Go to `http://localhost:5173`.
2.  **Connect**: Click **"Initialize Neural Link"**.
3.  **Permissions**: Allow Microphone access.
4.  **Visualize**: You should see the Green/Red status indicator turn **Active**.
5.  **Speak**: "This is a test of the emergency broadcast system."
6.  **Verify**: Text appears in the "Scribe" panel with a latency timer (e.g., `TAT: 1.2s`).

---

## 4. Running Tests

### Latency Benchmark
To verify if your CPU is fast enough:
1.  Watch the backend console.
2.  Speak a short phrase.
3.  Look for log: `Transcript generated in 0.45s`.
4.  **Target**: If > 2.0s, consider switching `model_size` to `tiny` in `main.py`.
