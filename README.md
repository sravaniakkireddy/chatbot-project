# RouterAI - Intermediate OpenRouter AI Chatbot

A clean, modern, and production-conscious intermediate-level AI Chatbot web application powered by **OpenRouter API**, built with **Node.js, Express.js, and Vanilla JavaScript (HTML5/CSS3)**.

Designed to demonstrate real-world full-stack architecture, secure API proxying, dynamic model discovery, multi-turn conversation memory, and zero-dependency XSS prevention.

---

## 📑 Table of Contents
- [Architecture Overview](#-architecture-overview)
- [Key Features](#-key-features)
- [Technology Stack](#-technology-stack)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Obtaining an OpenRouter API Key](#obtaining-an-openrouter-api-key)
  - [Installation & Environment Setup](#installation--environment-setup)
  - [Running the Application](#running-the-application)
- [API Endpoints Reference](#-api-endpoints-reference)
- [Security Architecture & Zero-Leak Guarantee](#-security-architecture--zero-leak-guarantee)
- [Testing Checklist](#-testing-checklist)
- [Common Troubleshooting](#-common-troubleshooting)
- [Future Enhancements](#-future-enhancements)

---

## 🏛 Architecture Overview

This project uses the **Backend-For-Frontend (BFF) / API Gateway Proxy Pattern**. The browser never communicates with OpenRouter directly. Instead, all traffic routes through our trusted Express backend, which injects the API key, enforces strict validation boundaries, sanitizes payloads, and isolates upstream exceptions.

```
┌─────────────────────────────────┐
│        Client (Browser)         │
│  Vanilla HTML5 / CSS3 / ES6 JS  │
│    (public/index.html & app.js) │
└───────────────┬─────────────────┘
                │ HTTP / REST (No credentials exposed)
                ▼
┌────────────────────────────────────────────────────────┐
│               Node.js / Express Backend                │
│  ├── Middleware: CORS, 1MB JSON Cap, Rate Limiter      │
│  ├── Routes: GET /api/test-key, GET /api/models,       │
│  │           POST /api/chat                            │
│  ├── Validator: Schema & character boundaries          │
│  └── Service: server/services/openrouter.js            │
└───────────────┬────────────────────────────────────────┘
                │ HTTPS Bearer Auth (Encrypted with .env Key)
                ▼
┌─────────────────────────────────┐
│         OpenRouter API          │
│  https://openrouter.ai/api/v1   │
│  (Claude, GPT-4o, LLaMA, etc.)  │
└─────────────────────────────────┘
```

---

## ✨ Key Features

1. **Secure API Key Testing (`GET /api/test-key`)**:
   - Queries OpenRouter's official `/auth/key` endpoint.
   - Detects missing, invalid, or expired keys.
   - Never returns the key to the frontend.

2. **Dynamic Model Discovery (`GET /api/models`)**:
   - Discovers 400+ real-time models dynamically from OpenRouter.
   - Normalizes pricing into readable `$ / 1M tokens`.
   - Automatically prioritizes and tags completely free models (`:free`).

3. **Intelligent Model Selection**:
   - Filter and search models by provider (OpenAI, Anthropic, Meta, Google, DeepSeek, etc.).
   - Model selection is remembered for the active browser session (`sessionStorage`).
   - No credentials ever touch browser storage.

4. **Multi-Turn Session Conversation**:
   - Full conversation memory during the active session.
   - User messages, assistant messages, and token usage statistics tracked per exchange.

5. **Production-Grade UI/UX**:
   - Modern dark mode styling with subtle borders, glowing status dots, and responsive cards.
   - Auto-growing prompt textarea.
   - Keyboard ergonomics: **Enter** to send, **Shift + Enter** for newline.
   - 1-click **Copy response** button with visual confirmation.
   - Interactive prompt suggestion cards.

6. **Defensive Security & XSS Prevention**:
   - Entity-level HTML escaping executed *prior* to markdown formatting.
   - Strict regex-based model ID validation.
   - IP rate-limiting protection via `express-rate-limit`.

---

## 🛠 Technology Stack

* **Frontend**: HTML5, CSS3 (Modern Flexbox & CSS Grid, Custom Properties), Vanilla JavaScript (ES6+ Modules, Fetch API).
* **Backend**: Node.js (v18+ native Fetch), Express.js.
* **Security & Utilities**: `dotenv`, `cors`, `express-rate-limit`.
* **API**: OpenRouter API (`/auth/key`, `/models`, `/chat/completions`).

---

## 📂 Project Structure

```
openrouter-chatbot/
│
├── server/
│   ├── server.js               # Application entry point, Express config, middleware & static hosting
│   ├── routes/
│   │   ├── api.js              # System endpoints: /api/health, /api/test-key, /api/models
│   │   └── chat.js             # Conversational completion endpoint: /api/chat
│   ├── services/
│   │   └── openrouter.js       # Centralized HTTP client for OpenRouter API with timeout handling
│   └── utils/
│       └── validator.js        # Schema validator for models and message payloads
│
├── public/
│   ├── index.html              # Semantic single-page application structure
│   ├── css/
│   │   └── style.css           # Modern dark-mode stylesheet & responsive design system
│   └── js/
│       └── app.js              # Client state, event listeners, XSS-safe markdown renderer
│
├── .env                        # Local secrets file (GIT-IGNORED)
├── .env.example                # Safe blueprint with dummy variables
├── .gitignore                  # Prevents secrets & node_modules from being committed
├── package.json                # Project manifest and scripts
└── README.md                   # Full documentation
```

---

## 🚀 Getting Started

### Prerequisites
* **Node.js**: v18.0.0 or higher (`node -v`)
* **npm**: v8.0.0 or higher (`npm -v`)

### Obtaining an OpenRouter API Key
1. Go to [https://openrouter.ai/](https://openrouter.ai/) and create an account.
2. Navigate to **Keys**: [https://openrouter.ai/keys](https://openrouter.ai/keys).
3. Click **Create Key**, give it a name, and copy the generated key (`sk-or-v1-...`).

### Installation & Environment Setup

1. **Clone or navigate to the project directory:**
   ```bash
   cd openrouter-chatbot
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Configure environment variables:**
   Copy `.env.example` to `.env`:
   ```bash
   # On Windows PowerShell:
   copy .env.example .env

   # On Linux/macOS:
   cp .env.example .env
   ```

4. **Add your OpenRouter key:**
   Open `.env` and replace `your_openrouter_api_key_here` with your key:
   ```env
   PORT=3000
   OPENROUTER_API_KEY=sk-or-v1-xxxxxxxxxxxxxxxxxxxxxxxxx
   SITE_URL=http://localhost:3000
   SITE_NAME=OpenRouter Chatbot
   ```

### Running the Application

* **Standard Start:**
  ```bash
  npm start
  ```
* **Development Mode (Auto-restart on save):**
  ```bash
  npm run dev
  ```

Open your browser and navigate to:
```
http://localhost:3000
```

---

## 📡 API Endpoints Reference

### 1. Health Check
* **Endpoint:** `GET /api/health`
* **Description:** Verifies server operational status and whether an API key is configured.
* **Response (200 OK):**
  ```json
  {
    "success": true,
    "status": "operational",
    "keyConfigured": true,
    "timestamp": "2026-09-28T13:45:00.000Z"
  }
  ```

### 2. Test API Key
* **Endpoint:** `GET /api/test-key`
* **Description:** Validates credentials upstream against `https://openrouter.ai/api/v1/auth/key`.
* **Response (200 OK):**
  ```json
  {
    "success": true,
    "message": "OpenRouter API key is valid",
    "data": {
      "label": "My Dev Key",
      "isFreeTier": false,
      "limit": null,
      "usage": 0.05
    }
  }
  ```

### 3. Dynamic Models Listing
* **Endpoint:** `GET /api/models`
* **Description:** Retrieves and normalizes models from OpenRouter.
* **Response (200 OK):**
  ```json
  {
    "success": true,
    "count": 458,
    "data": [
      {
        "id": "meta-llama/llama-3.3-70b-instruct:free",
        "name": "Meta: Llama 3.3 70B Instruct (free)",
        "provider": "META-LLAMA",
        "description": "...",
        "contextLength": 131072,
        "isFree": true,
        "pricing": {
          "promptPer1M": "0.00",
          "completionPer1M": "0.00"
        }
      }
    ]
  }
  ```

### 4. Chat Completion
* **Endpoint:** `POST /api/chat`
* **Headers:** `Content-Type: application/json`
* **Request Body:**
  ```json
  {
    "model": "meta-llama/llama-3.3-70b-instruct:free",
    "messages": [
      { "role": "user", "content": "Explain machine learning simply." }
    ]
  }
  ```
* **Response (200 OK):**
  ```json
  {
    "success": true,
    "model": "meta-llama/llama-3.3-70b-instruct:free",
    "message": {
      "role": "assistant",
      "content": "Machine learning is like teaching a computer to recognize patterns..."
    },
    "usage": {
      "promptTokens": 14,
      "completionTokens": 22,
      "totalTokens": 36
    }
  }
  ```

---

## 🔒 Security Architecture & Zero-Leak Guarantee

| Layer | Security Rule | Implementation |
| :--- | :--- | :--- |
| **Secrets Isolation** | Server-side only | `OPENROUTER_API_KEY` is loaded into memory via `dotenv` and injected into upstream requests inside `server/services/openrouter.js`. |
| **Version Control** | Never track secrets | `.env` is listed in `.gitignore`. Committed repository contains only `.env.example`. |
| **Browser Inspection** | Zero key transmission | Inspecting DevTools Network panel, Headers, or LocalStorage will reveal zero occurrences of the API key. |
| **Input Validation** | Defensive boundaries | Slashes/alphanumerics whitelist on `model`, 15,000-character cap on messages, 1MB maximum JSON body size. |
| **XSS Prevention** | Entity encoding | `escapeHtml()` escapes `<`, `>`, `&`, `"`, `'` before any markdown transformation occurs. |
| **Abuse Protection** | IP Rate limiting | `express-rate-limit` throttles traffic exceeding 120 calls per 15 minutes. |

---

## ✅ Testing Checklist

| # | Check Item | Expected Result | Pass / Fail |
| :-: | :--- | :--- | :-: |
| 1 | Server starts successfully | Console displays `🚀 OpenRouter Chatbot Server running on port 3000` | ✅ PASS |
| 2 | `.env` is loaded | `process.env.PORT` and `process.env.OPENROUTER_API_KEY` accessible | ✅ PASS |
| 3 | API key exists check | Server health check detects whether key is configured | ✅ PASS |
| 4 | API-key validation works | `GET /api/test-key` queries `/auth/key` and returns status 200 | ✅ PASS |
| 5 | Invalid API key is handled | Invalid key returns `{ success: false, message: "OpenRouter API key is invalid..." }` | ✅ PASS |
| 6 | Models retrieved dynamically | `GET /api/models` fetches live catalog from OpenRouter | ✅ PASS |
| 7 | Models displayed in UI | Dropdown populates with free and paid optgroups | ✅ PASS |
| 8 | Model selection works | Chosen model is saved in `sessionStorage` and attached to requests | ✅ PASS |
| 9 | User can send a message | Submitting form creates user bubble and disables submit | ✅ PASS |
| 10 | AI response appears | Assistant card appears with markdown, code blocks, and timestamp | ✅ PASS |
| 11 | Conversation history works | Multi-turn dialog retains context across multiple questions | ✅ PASS |
| 12 | Empty messages rejected | Send button disabled for empty input; backend validator rejects empty arrays | ✅ PASS |
| 13 | API errors displayed properly | Toast notification and in-chat error card appear for failures | ✅ PASS |
| 14 | Rate-limit errors handled | Upstream 429 returns clean guidance without breaking UI | ✅ PASS |
| 15 | DevTools secret leak test | Network response bodies and request headers contain zero API keys | ✅ PASS |
| 16 | Git secrets test | `.env` untracked by Git; `.gitignore` blocks all `.env*` files | ✅ PASS |

---

## 🛠 Common Troubleshooting

* **Q: I see "OpenRouter API key is missing or set to placeholder in .env"**
  * **Fix:** Open `.env` in your text editor and ensure you have placed your key in `OPENROUTER_API_KEY=sk-or-v1-...`. Restart the server (`npm start`).
* **Q: Model returns HTTP 402 "Insufficient credits"**
  * **Fix:** You selected a paid model while having a \$0 OpenRouter balance. In the dropdown, pick any model from the **⚡ Free Tier Models (`:free`)** section (e.g. `meta-llama/llama-3.3-70b-instruct:free`).
* **Q: Free model returns HTTP 429 "Rate limit reached"**
  * **Fix:** Free models on OpenRouter experience shared queue traffic. Wait 5-10 seconds, or select another free model from Google, Meta, or DeepSeek.

---

## 🔭 Future Enhancements

When you are ready to expand the application beyond intermediate level, consider:
* **Server-Sent Events (SSE) Streaming**: Stream tokens in real-time as they generate (`stream: true`).
* **Syntax Highlighting**: Integrate `Prism.js` or `highlight.js` for syntax-colored code blocks.
* **Persistent History**: Store conversations in SQLite / PostgreSQL / MongoDB.
* **Temperature & System Prompt Controls**: Sliders in UI to tune model creativity and custom system personalities.
* **Voice Input / TTS**: Web Speech API for voice dictation and synthesized speech playback.
* **Document Attachment / RAG**: Upload PDF/text files with embeddings for document question-answering.
"# chatbot-project" 
