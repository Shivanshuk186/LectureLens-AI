LectureLens AI — Setup Guide

Run LectureLens AI on your own machine.

No GPU, transcription, Qdrant account, or full re-embedding is required for the prebuilt-index workflow. The repository is expected to include 126 transcripts and a prebuilt search index. You need one supported API key and a few minutes for setup.

Setup

1. Install

git clone <REPO_URL>
cd <YOUR_PROJECT_DIRECTORY>
uv sync

If you do not have uv, install it first with pip install uv.

2. Configure one API key

cp .env.example .env

Open .env and configure the provider you intend to use:

Gemini: https://aistudio.google.com/apikey — set GEMINI_API_KEY=

Groq: https://console.groq.com/keys — set GROQ_API_KEY=

Use the provider supported by your current project configuration. Do not paste API keys into source code or commit .env to Git.

The default local-index workflow does not require a Qdrant account if the project is configured to use its local index.

3. Load the prebuilt index

uv run ytrag load

This command is expected to load the prebuilt index from index/vectors.npz into the local vector store. It does not re-embed the corpus. The query embedding model may download on first use.

If the prebuilt index is missing or this command is not available in your checked-out version, use the troubleshooting section below and verify the commands against uv run ytrag --help.

4. Start the web app

uv run ytrag serve

Open http://127.0.0.1:8000 in your browser. Ask a question and select a timestamp citation to jump to the relevant lecture moment.

Every time after setup

Start the app with:

uv run ytrag serve

The first startup may take longer while the model loads. Wait until the terminal reports that the app is ready.

Press Ctrl+C to stop the server. The local index remains on disk.

CLI commands that do not require the web server:

uv run ytrag search "sliding window"       # timestamps only; no LLM answer
uv run ytrag ask "kadane ka intuition"     # generate a written answer
uv run ytrag stats

Run only one process against the local index at a time. Stop ytrag serve before running another command that accesses the same local index. A hosted vector store may allow a different workflow, depending on your configuration.

Rebuilding the index

To rebuild the index from cached transcripts:

uv run ytrag reindex

This can take longer than loading the prebuilt index. Reindex if you change chunking settings or want to evaluate a different embedding model.

Choosing an embedding model

The supplied setup notes report these measurements for a corpus of 2,933 chunks:

Model

Download

CPU reindex time

Top-1

Top-5

all-MiniLM-L6-v2 (default in the supplied setup notes)

87 MB

1.6 min

11/12

12/12

BAAI/bge-m3

4.35 GB

55 min

12/12

12/12

These are project-reported measurements, not independently verified benchmarks. BAAI/bge-m3 may improve multilingual retrieval, but requires a much larger download and longer CPU reindexing time.

To try it, if supported by your project configuration:

YTRAG_EMBED_MODEL=BAAI/bge-m3 uv run ytrag reindex

Troubleshooting

Run the preflight check:

uv run ytrag preflight

It should check the configured dependencies, vector store, embedding model, and LLM provider.

A warning about unauthenticated Hugging Face Hub requests is generally about download rate limits. A Hugging Face token is not necessarily required for public models.

Symptom

Possible fix

GROQ_API_KEY is not set

Check that .env exists in the project root and contains the key for the configured provider.

Vector/index embedding mismatch

Check YTRAG_EMBED_MODEL; use the model expected by the supplied index or rebuild the index.

Index is already open in another process

Stop ytrag serve before running another command against the local index.

No prebuilt index found

Confirm you are running commands from the project root and that the index files are present; otherwise run uv run ytrag reindex if transcripts are available.

LLM quota errors

Check the provider's quota/configuration. Search may work without an LLM if your project supports that mode.

If a command is unavailable in your version, run:

uv run ytrag --help

Use the commands listed by your installed project version.

Use your own playlist

Ingesting a new playlist requires downloading and transcribing its videos, so it takes significantly longer than using the prebuilt index. Hardware and runtime vary.

uv run ytrag langtest "<ONE_VIDEO_URL>"
uv run ytrag ingest --playlist "<PLAYLIST_URL>" --limit 10

Test the transcription language on one video before ingesting a full playlist. Start with a small limit, verify the results, and then ingest more. Review the main README for additional details about caching, timestamps, and long-running jobs.