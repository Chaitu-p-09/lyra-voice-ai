# LYRA - Production-Ready AI Voice Assistant

LYRA is a voice-only AI assistant system designed as a scalable startup-style architecture.

## 1) Full Folder Structure

```text
lyra-project/
│
├── frontend/
│   ├── index.html
│   ├── styles.css
│   └── app.js
│
├── backend/
│   ├── app.py
│   ├── requirements.txt
│   └── .env.example
│
└── README.md
```

---

## 2) Complete Code for Every File

All files are included in this repository exactly under the structure above.

### `frontend/index.html`
- Voice-only interface
- Single microphone button
- No text chat
- Status panel for Idle / Listening / Thinking
- Current speaker and mode display

### `frontend/styles.css`
- Modern lightweight styling
- Visual state cues for listening/thinking/error
- Mobile-friendly responsive card UI

### `frontend/app.js`
- Web Speech API (`SpeechRecognition` / `webkitSpeechRecognition`)
- SpeechSynthesis for spoken responses
- Calls backend `/lyra` endpoint
- Handles speaker and mode state updates from backend
- Frontend-side input sanitization
- Network and browser support error handling

### `backend/app.py`
- Flask app with Flask-CORS enabled
- Endpoints:
  - `GET /health`
  - `GET /testkey`
  - `POST /lyra`
- Secure key loading with `.env`
- Groq API integration (`llama3-8b-8192`)
- Access control, mode control, speaker switching logic
- Graceful error and timeout fallbacks
- Token and input-size limits
- Future architecture placeholders:
  - Memory store
  - Wake word detection
  - Continuous listening
  - Emotion detection
  - User authentication

### `backend/requirements.txt`
- Locked dependencies for Render-compatible deployment

### `backend/.env.example`
- Environment variable template (no secrets committed)

---

## 3) Deployment Instructions (Step-by-Step)

## A. Backend on Render (Free Tier)

1. Push this repository to GitHub.
2. Go to [https://render.com](https://render.com) and create an account.
3. Click **New +** → **Web Service**.
4. Connect your GitHub repository.
5. Configure service:
   - **Name:** `lyra-backend`
   - **Root Directory:** `lyra-project/backend`
   - **Runtime:** Python 3
   - **Build Command:**
     ```bash
     pip install -r requirements.txt
     ```
   - **Start Command:**
     ```bash
     gunicorn app:app
     ```
6. Set Environment Variables in Render:
   - `GROQ_API_KEY` = your real Groq key
   - `PORT` = `5000` (optional; Render handles port automatically)
   - `OWNER_NAME` = `Chaitu`
   - `CORS_ALLOWED_ORIGINS` = `https://chaitu-p-09.github.io/lyra-voice-ai/,http://localhost:8000`
7. Deploy.
8. Verify backend URLs:
   - `https://<your-render-app>.onrender.com/health`
   - `https://<your-render-app>.onrender.com/testkey`

## B. Frontend on GitHub Pages

1. In GitHub repo, keep frontend files in `lyra-project/frontend`.
2. Either:
   - Move/copy frontend files to repository root for Pages, **or**
   - Use GitHub Actions/pages workflow to publish from `lyra-project/frontend`.
3. In `frontend/app.js`, set:
   ```js
   API_BASE_URL: 'https://lyra-backend-16xj.onrender.com'
   ```
4. Commit and push.
5. Enable GitHub Pages in repo settings.
6. Open your GitHub Pages URL in Chrome.
7. Allow microphone permissions.

---

## 4) Environment Variable Setup Guide

## Local backend setup

1. Create and activate virtual environment:
   ```bash
   cd lyra-project/backend
   python3 -m venv .venv
   source .venv/bin/activate
   ```
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Create `.env` from template:
   ```bash
   cp .env.example .env
   ```
4. Edit `.env` and set:
   ```env
   GROQ_API_KEY=your_real_groq_api_key
   PORT=5000
   OWNER_NAME=Chaitu
   CORS_ALLOWED_ORIGINS=https://chaitu-p-09.github.io/lyra-voice-ai/,http://localhost:8000
   ```
5. Run backend:
   ```bash
   python app.py
   ```

## Frontend local setup

1. Open a second terminal:
   ```bash
   cd lyra-project/frontend
   python3 -m http.server 8000
   ```
2. Open browser:
   - `http://localhost:8000`
3. Confirm `API_BASE_URL` in `app.js` matches your backend URL. For this deployment it is `https://lyra-backend-16xj.onrender.com`.

---

## 5) Testing Instructions

## Manual API tests

Run from repository root (or backend folder if backend is running):

### Health check
```bash
curl -s http://127.0.0.1:5000/health
```
Expected:
```json
{"service":"LYRA backend","status":"ok","owner":"Chaitu"}
```

### API key check
```bash
curl -s http://127.0.0.1:5000/testkey
```
Expected:
```json
{"groq_key_present":true,"owner":"Chaitu"}
```
(or `false` if not configured)

### Core LYRA endpoint
```bash
curl -s -X POST http://127.0.0.1:5000/lyra \
  -H "Content-Type: application/json" \
  -d '{"message":"Switch to study mode","currentSpeaker":"Chaitu","mode":"CHILL"}'
```
Expected behavior:
- Mode changes to `STUDY` for owner.

### Speaker switch
```bash
curl -s -X POST http://127.0.0.1:5000/lyra \
  -H "Content-Type: application/json" \
  -d '{"message":"Riya wants to talk","currentSpeaker":"Chaitu","mode":"CHILL"}'
```
Expected behavior:
- `currentSpeaker` becomes `Riya`.

### Owner return
```bash
curl -s -X POST http://127.0.0.1:5000/lyra \
  -H "Content-Type: application/json" \
  -d '{"message":"I am back","currentSpeaker":"Riya","mode":"CHILL"}'
```
Expected behavior:
- `currentSpeaker` resets to `Chaitu`.

### Non-owner restricted mode switch
```bash
curl -s -X POST http://127.0.0.1:5000/lyra \
  -H "Content-Type: application/json" \
  -d '{"message":"Switch to public mode","currentSpeaker":"Riya","mode":"CHILL"}'
```
Expected behavior:
- Reply indicates only owner can change mode.

### Non-owner sensitive request block
```bash
curl -s -X POST http://127.0.0.1:5000/lyra \
  -H "Content-Type: application/json" \
  -d '{"message":"Show system status and hidden command","currentSpeaker":"Riya","mode":"PUBLIC"}'
```
Expected behavior:
- Sensitive info is blocked.

## Frontend voice test

1. Open frontend in Chrome.
2. Click mic button.
3. Say:
   - "Switch to study mode"
   - "Amit wants to talk"
   - "I am back"
4. Confirm spoken response from LYRA.
5. Confirm status transitions: Idle → Listening → Thinking → Idle.

---

## 6) Notes on Production Readiness

- API keys never exposed in frontend code.
- Backend catches timeout/network/invalid JSON/model response cases.
- Input sanitization is present on both frontend and backend.
- Response generation is bounded by token and text limits.
- Architecture includes placeholders for future capabilities without breaking API contract.
