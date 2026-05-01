# Thesis Monitor

A lightweight investment thesis management tool built with Flask and SQLite. Commit structured investment theses, run AI-powered delta checks against live web sources and uploaded documents, and maintain a versioned research trail over time.

---

## What it does

**Thesis Commit** — Structure an investment thesis with variant perception, thesis drivers, scenario probabilities, kill-switches, and a watchlist. Import from a PDF or pasted text and let Groq auto-fill the fields.

**Delta Check** — Run a delta check against the latest thesis. The app generates exchange-aware search queries (LSE, NYSE, SGX, HKEX, ASX), fetches results via Tavily, optionally incorporates uploaded PDFs via BM25 RAG, and asks Groq to assess materiality against the thesis. Accept a suggested update to automatically synthesise a new thesis version.

**Research Trail** — A chronological log of every thesis commit and delta check, with full evidence, search queries, sources, and decision notes. Exportable as JSON.

**Feedback** — Users can submit feedback with a star rating and category. Responses are stored in a separate `feedback.db` and downloadable as CSV at any time via `/api/feedback/export`.

---

## Stack

| Component | Technology |
|---|---|
| Backend | Python 3.10+, Flask 3.1 |
| LLM | Groq (`meta-llama/llama-4-scout-17b-16e-instruct`) |
| Web search | Tavily Python SDK |
| PDF extraction | pypdf |
| Document retrieval | BM25 (rank-bm25) |
| Live prices | Yahoo Finance (no API key required) |
| Database | SQLite (`thesis.db` + `feedback.db`) |
| Frontend | Vanilla JS, single-file HTML template |
| Production server | Gunicorn |

---

## Getting started locally

### 1. Clone and install

```bash
git clone <your-repo-url>
cd PRICE-Guard
pip install -r requirements.txt
```

### 2. Set environment variables

Copy `.env.example` to `.env` and fill in your keys:

```bash
cp .env.example .env
```

```env
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key
```

Get your keys at:
- Groq: https://console.groq.com
- Tavily: https://tavily.com

### 3. Run

```bash
python app.py
```

Open `http://localhost:5000` in your browser.

---

## Environment variables

| Variable | Default | Description |
|---|---|---|
| `GROQ_API_KEY` | — | Required. Groq API key for LLM calls. |
| `TAVILY_API_KEY` | — | Required for web search in delta checks. |
| `DB_PATH` | `thesis.db` | Path to the main SQLite database. |
| `FEEDBACK_DB_PATH` | `feedback.db` | Path to the feedback SQLite database. |
| `PORT` | `5000` | Port for the Flask/Gunicorn server. |

---

## Deploying to Render

### Free tier

1. Push the repo to GitHub.
2. Create a new **Web Service** on [render.com](https://render.com), connect the repo.
3. Set the build command to `pip install -r requirements.txt`.
4. Set the start command to `gunicorn app:app`.
5. Add `GROQ_API_KEY` and `TAVILY_API_KEY` as environment variables in the Render dashboard.

> **Note:** Render's free tier uses ephemeral storage. `thesis.db` and `feedback.db` are reset on every redeploy or instance restart. Download your research trail JSON and feedback CSV before redeploying.

### Persistent storage (paid)

Add a **Persistent Disk** in Render and mount it at `/data`. Then set:

```
DB_PATH=/data/thesis.db
FEEDBACK_DB_PATH=/data/feedback.db
```

Data will survive restarts and redeploys.

---

## Collecting feedback

Visit `/api/feedback/export` in your browser to download all feedback as a CSV file. A response count is shown on the Feedback tab so you know when it's worth downloading.

---

## Project structure

```
.
├── app.py                  # Flask backend — all routes and business logic
├── templates/
│   └── index.html          # Single-page frontend (vanilla JS)
├── requirements.txt
├── .env                    # Local secrets (not committed)
├── .env.example            # Template for environment variables
├── thesis.db               # SQLite — thesis commits and delta checks (auto-created)
└── feedback.db             # SQLite — user feedback (auto-created)
```

---

## Key API endpoints

| Method | Path | Description |
|---|---|---|
| `POST` | `/api/thesis` | Commit a new thesis version |
| `GET` | `/api/thesis/latest` | Get the most recent thesis |
| `POST` | `/api/parse-document` | Parse a PDF/text into thesis fields via Groq |
| `POST` | `/api/delta-check` | Run a delta check against the latest thesis |
| `POST` | `/api/delta-check/<id>/accept-and-update` | Accept a suggested update and synthesise a new thesis version |
| `GET` | `/api/trail` | Get the full research trail |
| `GET` | `/api/trail/export` | Download the trail as JSON |
| `POST` | `/api/import-trail` | Restore a previously exported trail |
| `GET` | `/api/fetch-price/<ticker>` | Fetch live price from Yahoo Finance |
| `POST` | `/api/feedback` | Submit user feedback |
| `GET` | `/api/feedback/export` | Download all feedback as CSV |
| `GET` | `/api/feedback/count` | Get number of feedback responses |
