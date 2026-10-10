<div align="center">

# LectureLens AI

### Every Concept. Every Timestamp. One Search Away.

**An AI-powered search engine for DSA lectures.** Ask questions in English or Hinglish, get grounded answers, and jump directly to the relevant moment in a YouTube lecture.

<br>

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-API-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![Qdrant](https://img.shields.io/badge/Qdrant-Vector_Search-DC244C?style=for-the-badge)
![Groq](https://img.shields.io/badge/Groq-LLM-F55036?style=for-the-badge)
![Status](https://img.shields.io/badge/Project-Active-success?style=for-the-badge)

</div>

---

## Contents

- [What is LectureLens AI?](#-what-is-lecturelens-ai)
- [How it works](#-how-it-works)
- [Features](#-features)
- [Technology stack](#-technology-stack)
- [Quick start](#-quick-start)
- [Usage](#-usage)
- [Configuration](#-configuration)
- [Evaluation](#-evaluation)
- [Project structure](#-project-structure)
- [Important implementation details](#-important-implementation-details)
- [Deployment notes](#-deployment-notes)

---

## 🎯 What is LectureLens AI?

Finding one explanation inside hours of lectures can be frustrating. LectureLens AI makes lecture content searchable: ask a question and receive a concise answer supported by relevant lecture segments and timestamps.

For example:

> **Question:** “Bhai, memoization aur tabulation ka difference kahan explain kiya hai?”
>
> **Result:** A short, context-grounded answer with source links such as:
>
> - **DP Lecture 3 — Memoization Deep Dive** · `12:04`
> - **DP Lecture 5 — Tabulation Recipe** · `04:31`

Select a citation to jump to that moment in the embedded YouTube player.

> **Core idea:** Instead of returning only a block of text, LectureLens AI helps you find *where a concept was explained*.

---

## 🔄 How it works

```text
YouTube playlist
      │
      ▼
Download audio ──► Transcribe with faster-whisper
                         │
                         ▼
                  Time-aware chunking
                         │
                         ▼
                 Generate embeddings
                         │
                         ▼
                  Qdrant vector search
                         │
                         ▼
Question ──► Retrieve relevant lecture segments
                         │
                         ▼
                 Groq-powered answer
                         │
                         ▼
          Answer + timestamped citations
```

The retrieval pipeline preserves timestamps so answers can point back to the source lecture. The system also includes relevance checks to reduce unsupported answers.

---

## ✨ Features

- **Natural-language search:** Ask questions in English or Hinglish.
- **Timestamped citations:** Jump directly to relevant moments in a YouTube lecture.
- **Retrieval-augmented answers:** Answers are based on retrieved lecture context.
- **Time-aware chunking:** Keeps lecture segments associated with their timestamps.
- **Cached transcription:** Reuse completed transcripts instead of repeatedly transcribing videos.
- **Resumable ingestion:** Continue after interrupted runs without redoing all completed work.
- **Hybrid-ready evaluation workflow:** Maintain a golden set of questions to assess retrieval and refusal behavior.
- **Web interface and CLI:** Search through the browser or run commands from the terminal.
- **Configurable pipeline:** Adjust transcription, chunking, embeddings, retrieval, and model settings through environment variables.

---

## 🧰 Technology stack

| Layer | Technology |
|---|---|
| Language | Python |
| Playlist and audio | `yt-dlp` |
| Speech-to-text | `faster-whisper` with CTranslate2 |
| Chunking | Custom, timestamp-aware chunking |
| Embeddings | `sentence-transformers` with `BAAI/bge-m3` |
| Vector database | Qdrant Cloud |
| Language model | Groq — `openai/gpt-oss-120b` |
| CLI | Typer and Rich |
| API | FastAPI |
| Web UI | HTML and YouTube IFrame API |
| Environment and dependencies | `uv` |

---

## 🚀 Quick start

### Prerequisites

- Python 3.11
- [`uv`](https://docs.astral.sh/uv/)
- A Groq API key
- A Qdrant Cloud URL and API key
- CUDA-compatible GPU recommended for full-playlist transcription; CPU fallback is available but slower

### 1. Clone the repository

```bash
git clone https://github.com/Shivanshuk186/LectureLens-AI.git
cd LectureLens-AI
```

### 2. Create the environment and install dependencies

```bash
uv venv --python 3.11
uv sync
```

For the optional CUDA runtime dependencies used by CTranslate2 on Windows:

```bash
uv sync --extra cuda
```

### 3. Configure environment variables

Create a `.env` file in the repository root. Use `.env.example` as the template and add your own credentials:

```dotenv
GROQ_API_KEY=your_groq_api_key
QDRANT_URL=your_qdrant_url
QDRANT_API_KEY=your_qdrant_api_key
```

Keep `.env` private. **Never commit API keys or other secrets.**

### 4. Run preflight checks

```bash
uv run ytrag preflight
```

To check a specific playlist as well:

```bash
uv run ytrag preflight --playlist "<PLAYLIST_URL>"
```

The preflight command checks key dependencies and pipeline components before a long ingestion run.

---

## 💻 Usage

### Start the web interface

```bash
uv run ytrag serve
```

Open **http://127.0.0.1:8000** in your browser.

The interface embeds the YouTube player. Selecting a timestamped citation seeks to the relevant point in the video.

### Ask a question from the CLI

```bash
uv run ytrag ask "memoization aur tabulation ka difference kya hai?"
```

### Search without generating an answer

```bash
uv run ytrag search "adjacency list"
```

This displays retrieved matches and their distances, helping you inspect retrieval quality.

### Ingest a playlist

First test with one video:

```bash
uv run ytrag ingest --playlist "<PLAYLIST_URL>" --limit 1
```

Then ingest the complete playlist:

```bash
uv run ytrag ingest --playlist "<PLAYLIST_URL>"
```

The pipeline downloads audio, transcribes it, caches the transcript, creates timestamp-aware chunks, and uploads the resulting vectors. Completed work is designed to be reusable on subsequent runs.

**Before a long run:** use `langtest` to compare transcription language settings and inspect how technical terms are recognized.

```bash
uv run ytrag langtest "https://www.youtube.com/watch?v=<VIDEO_ID>"
```

### Rebuild the index from cached transcripts

```bash
uv run ytrag reindex
```

If chunking settings have changed and old chunks must be removed first:

```bash
uv run ytrag reindex --replace
```

### Evaluate retrieval

```bash
uv run ytrag eval --verbose
```

### Other useful commands

```bash
uv run ytrag stats
uv run ytrag clean-audio
uv run ytrag preflight
```

---

## ⚙️ Configuration

Configuration defaults are defined in [`ytrag/config.py`](ytrag/config.py). Set the variables in your root `.env` file to customize behavior.

| Variable | Default | Purpose |
|---|---|---|
| `YTRAG_WHISPER_LANG` | `en` | Transcription language; validate with `langtest` |
| `YTRAG_WHISPER_MODEL` | `large-v3` | Whisper model; `medium` is a lighter option |
| `YTRAG_WHISPER_DEVICE` | `auto` | Device selection: `auto`, `cuda`, or `cpu` |
| `YTRAG_WHISPER_BATCH` | `8` | Transcription batch size; `0` disables batching |
| `YTRAG_WHISPER_BEAM` | `5` | Beam size; `1` is faster but uses greedy decoding |
| `YTRAG_CHUNK_SECONDS` | `75` | Target chunk duration in seconds |
| `YTRAG_CHUNK_OVERLAP` | `15` | Overlap between adjacent chunks, in seconds |
| `YTRAG_EMBED_MODEL` | `BAAI/bge-m3` | Embedding model; `all-MiniLM-L6-v2` is a smaller alternative |
| `YTRAG_COLLECTION` | `dsa_lectures` | Base Qdrant collection name |
| `YTRAG_TOP_K` | `6` | Number of retrieved candidates |
| `YTRAG_MAX_DISTANCE` | `0.5` | Relevance cutoff; tune using real queries and evaluation |
| `YTRAG_LLM_MODEL` | `openai/gpt-oss-120b` | Groq model identifier |

Use the exact variable names expected by the current code. Changing embedding models may require rebuilding the index because embedding dimensions can differ.

---

## 🧪 Evaluation

The evaluation questions live in [`eval/golden.json`](eval/golden.json).

The set supports two types of checks:

- **Retrieval checks:** Whether the expected lecture appears among the top results and a retrieved chunk falls near the expected timestamp.
- **Refusal checks:** Whether the system declines to answer questions outside the indexed lecture content.

Run:

```bash
uv run ytrag eval --verbose
```

Add representative questions from your own playlist and verify both successful retrievals and out-of-scope refusals. Treat relevance thresholds as starting points and validate them against the complete index.

---

## 🗂️ Project structure

```text
LectureLens-AI/
├── api/
│   ├── main.py
│   └── static/
│       └── index.html
├── eval/
│   ├── golden.json
│   └── golden_draft.json
├── index/
│   └── vectors.npz
├── ytrag/
│   ├── answer.py
│   ├── chunk.py
│   ├── cli.py
│   ├── config.py
│   ├── embed.py
│   ├── evaluate.py
│   ├── index.py
│   ├── models.py
│   ├── playlist.py
│   ├── transcribe.py
│   └── util.py
├── .env.example
├── .gitignore
├── main.py
├── pyproject.toml
├── uv.lock
├── README.md
├── SETUP.md
└── RAGPLAN.md
```

Local transcripts and environment secrets are intentionally excluded from version control. Check `.gitignore` before adding local data.

---

## 🧠 Important implementation details

### Timestamp-aware chunking

Generic text splitting can separate an explanation from its timestamp. LectureLens AI chunks the transcript in the time domain, preserving the association between text and video position.

### Transcript caching and resumability

Transcripts are cached so completed videos do not need to be transcribed again. Atomic writes help avoid leaving partially written cache files after interruption.

### Relevance checks and grounded answers

Retrieval uses a distance cutoff before context is sent to the language model. If no suitable context survives, the system can refuse to answer instead of inventing an explanation with misleading citations. Answers without citations are not treated as grounded.

### Reliable ingestion

The ingestion pipeline is designed to tolerate interrupted runs, retry transient failures, and avoid duplicate vector entries through deterministic IDs and upserts.

### Transcription performance

The project uses batched `faster-whisper` inference. Actual throughput depends on GPU, drivers, model configuration, audio, and batch settings; measure performance on your own machine before estimating the time for a full playlist.

---

## ☁️ Deployment notes

The FastAPI application is exposed as `api.main:app`. A deployment platform such as Render, Railway, or Fly.io can host the API, provided the required environment variables and model/vector services are configured.

Before deploying:

1. Configure the Groq and Qdrant credentials as platform secrets.
2. Confirm the selected embedding model and Qdrant collection dimensions match.
3. Test `/ask` and the web UI locally.
4. Check the platform's memory, request timeout, and concurrency limits.
5. If running multiple API workers, use a shared rate limiter rather than relying on in-process state.

The full transcription pipeline is resource-intensive. For a hosted demo, prepare the index before deployment instead of transcribing the entire playlist during startup.

---

## 🔐 Privacy and repository hygiene

- Store credentials in `.env`; do not commit that file.
- Keep `.env.example` limited to blank placeholders.
- Keep local transcript files excluded if you do not intend to redistribute them.
- Confirm that you have permission to index and redistribute any lecture-derived data you publish.
- Review the repository contents before each push.

---

## 📌 Project status

LectureLens AI is a lecture-search project focused on grounded answers, useful citations, and fast navigation to the relevant point in a video.

**Every Concept. Every Timestamp. One Search Away.**

<div align="center">

Built with Python · FastAPI · Qdrant · Groq

</div>
