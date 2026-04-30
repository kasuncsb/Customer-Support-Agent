<div align="center">
  <img src="assets/animated-orb.svg" width="160" height="160" alt="Animated orb logo" />
  <h1>Customer Support Agent</h1>
  <p><strong>LankaTel AI Support Assistant</strong></p>
  <p>RAG-first support with voice, agentic actions, and a modern UI.</p>
  <p>
    <a href="https://www.python.org/downloads/"><img src="https://img.shields.io/badge/python-3.10+-blue.svg" alt="Python 3.10+" /></a>
    <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="MIT License" /></a>
    <a href="https://github.com/psf/black"><img src="https://img.shields.io/badge/code%20style-black-000000.svg" alt="Code style: black" /></a>
  </p>
</div>

A RAG-based AI customer support agent for LankaTel — a conceptual ISP in Sri Lanka. Built with Azure OpenAI and ChromaDB, it ships a FastAPI backend, a Next.js UI, and voice + agentic account workflows out of the box.

---

## Features

- **RAG with citations** — retrieves relevant chunks and surfaces source references
- **Agentic account actions** — ticket creation, subscription changes, and balance queries backed by MySQL
- **OTP verification** — mobile OTP shared across chat and voice sessions; unlocks a quick-actions panel
- **Voice modes** — browser STT + Azure TTS in the UI; full-duplex realtime voice in the CLI
- **Admin console** — JWT-protected management UI
- **Single-container deploy** — Docker image with nginx reverse-proxying both the Next.js app and the Python API

---

## Tech Stack

| Layer | Technology |
|---|---|
| **AI / RAG** | Azure OpenAI (GPT-4o-mini, text-embedding-3-large), ChromaDB, tiktoken |
| **Backend** | Python 3.10+, FastAPI, Uvicorn, Pydantic v2, SQLAlchemy + PyMySQL |
| **Frontend** | Next.js 16, React 19, TypeScript, Tailwind CSS |
| **Database** | MySQL 8.x |
| **Voice** | Browser Web Speech API (STT), Azure Speech Services (TTS + realtime STT/TTS) |
| **Email** | Zoho Mail SMTP (OTP / notifications) |
| **Ops** | Docker + nginx, pytest, ESLint, Prettier |

---

## Architecture

```
[Next.js UI] ──/api──► [FastAPI backend]
                              ├── [RAG pipeline] ──► [ChromaDB]
                              ├── [Azure OpenAI]
                              ├── [MySQL]  (actions, sessions, admin)
                              └── [Azure Speech]  (voice + TTS)
```

---

## Project Structure

```
Customer-Support-Agent/
  api_server.py          # FastAPI backend entry point
  src/                   # Core pipeline, services, realtime voice
  ui/                    # Next.js frontend
  data/
    raw/                 # Raw JSONL knowledge files
    processed/           # Generated KB (gitignored)
  db/                    # MySQL schema + seed data
  docker/                # nginx config + container entrypoint
  scripts/               # Utilities (KB build, quota check)
  tests/                 # pytest suite
  vectorstore/           # ChromaDB persistence (gitignored)
```

---

## Quick Start

### Prerequisites

- Python 3.10+
- Node.js 18+
- Azure OpenAI deployments: `chat-model` (GPT-4o-mini) and `embedding-model` (text-embedding-3-large)
- MySQL 8.x _(optional — required for account actions and admin console)_
- Azure Speech Services _(optional — required for voice features)_

### 1. Backend setup

```bash
python -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt

cp .env.example .env              # fill in your Azure OpenAI credentials
python -m src.cli test            # verify configuration
```

### 2. Database setup (optional)

```bash
mysql -u root -p < db/mysql_schema.sql
# Set DB_URL in .env to: mysql+pymysql://user:password@localhost/ltagent
```

### 3. Ingest knowledge base

```bash
python -m src.cli ingest data/raw/lankatel   # bundled Lankatel data
python -m src.cli ingest data/documents/ --recursive   # custom documents
```

### 4. Frontend setup

```bash
cd ui
cp .env.example .env.local        # default: BACKEND_URL=http://localhost:8000
npm install
```

### 5. Run

```bash
# Terminal 1 — backend
python api_server.py

# Terminal 2 — frontend
cd ui && npm run dev
```

- UI: `http://localhost:3000`
- API: `http://localhost:8000`

---

## CLI Reference

```bash
python -m src.cli <command>
```

| Command | Description |
|---|---|
| `ingest <path>` | Ingest documents from a file or directory |
| `query <question>` | Ask a single question |
| `chat` | Interactive chat session |
| `voice-chat` | Turn-based voice chat (Azure Speech) |
| `realtime` | Full-duplex realtime voice chat |
| `stats` | Show system statistics |
| `test` | Verify system configuration |
| `clear` | Clear the vector store |

---

## Voice Modes

**UI voice** — uses the browser Web Speech API for STT and the backend `/api/tts` endpoint (Azure Speech) for TTS. Supports OTP verification and agentic actions by voice.

**CLI voice-chat** — turn-based speech via Azure Speech SDK.

**CLI realtime** — low-latency full-duplex voice with barge-in support.

---

## Verification & Agentic Actions

Account actions (ticket creation, subscription changes, balance queries) require OTP verification:

1. Enter your mobile number in the chat or voice UI.
2. Enter the OTP received via SMS.
3. A verified session is shared across chat and voice — a quick-actions panel appears once verified.
4. Actions run on non-streaming responses to keep tool calls deterministic.

---

## Data Formats

| Format | Extension | Notes |
|---|---|---|
| Text | `.txt` | Plain text |
| Markdown | `.md` | Markdown documents |
| JSON | `.json` | Object with `text` or `content` field |
| JSON Lines | `.jsonl` | One JSON object per line |

JSONL example:

```json
{"id": "faq_001", "question": "How do I reset my password?", "answer": "Click 'Forgot Password'...", "category": "account"}
{"id": "faq_002", "question": "What payment methods?", "answer": "We accept Visa, MasterCard...", "category": "billing"}
```

To rebuild the processed Lankatel KB manually:

```bash
python scripts/build_processed_kb.py
```

---

## Configuration

All settings are controlled by environment variables. Copy `.env.example` to `.env` (backend) and `ui/.env.example` to `ui/.env.local` (frontend). Key variables:

```bash
# Azure OpenAI (required)
AZURE_OPENAI_API_KEY=
AZURE_OPENAI_ENDPOINT=
AZURE_OPENAI_CHAT_DEPLOYMENT=chat-model
AZURE_OPENAI_EMBEDDING_DEPLOYMENT=embedding-model

# Vector store
VECTORSTORE_DIR=./vectorstore
VECTORSTORE_COLLECTION=support_docs

# Database (optional)
DB_URL=mysql+pymysql://user:password@localhost/ltagent

# Voice (optional)
AZURE_SPEECH_API_KEY=
AZURE_SPEECH_REGION=eastus

# Email / OTP (optional)
EMAIL_ENABLED=false
EMAIL_SMTP_HOST=smtp.zoho.com
EMAIL_SMTP_PORT=587
EMAIL_SMTP_USERNAME=
EMAIL_SMTP_PASSWORD=
EMAIL_SENDER=

# Admin console
ADMIN_USERNAME=ltadmin
ADMIN_PASSWORD=change_me_strong
ADMIN_SECRET=change_me_secret
```

---

## Testing

```bash
pytest tests/ -v
```

---

## Docker

Builds the Next.js UI and packages the Python backend into a single image. nginx reverse-proxies `/api/*` to uvicorn on port 8000 and everything else to Next.js on port 3000.

```bash
docker build -t customer-support-agent:latest .
docker run -p 80:80 customer-support-agent:latest
```

**Azure deployment:** push the image to ACR or Docker Hub, then point an Azure Web App for Containers or Azure Container Instance at it. The container exposes port 80 via nginx, which is what Azure expects.

---

## Contributing

Open a PR with a clear description of the change.

## License

MIT — see [LICENSE](LICENSE).

## Acknowledgments

- [LangChain](https://github.com/langchain-ai/langchain) for RAG patterns
- [ChromaDB](https://www.trychroma.com/) for vector storage
- [Azure OpenAI](https://azure.microsoft.com/en-us/products/ai-services/openai-service) for LLMs and embeddings
