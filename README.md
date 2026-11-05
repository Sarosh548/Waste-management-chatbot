# Waste-Management RAG Chatbot

A production-grade retrieval-augmented generation (RAG) chatbot built for a government-level waste management simulator. It answers technical questions from a corpus of engineering textbooks — and backs every answer with a citation you can check.

## Overview

General-purpose chatbots will happily answer a technical waste-management question from memory, whether or not the answer is right. This chatbot is constrained to a defined knowledge base instead: 6 engineering textbooks, totalling 3,765 pages, indexed as 13,000+ chunks. Answers are retrieved from that corpus first, generated second, and returned with book, chapter and page references.

## Features

- **Large, real corpus** — technical knowledge retrieval across 6 engineering textbooks (3,765 pages, 13,000+ chunks)
- **Hybrid retrieval pipeline** — ChromaDB combined with Sentence-Transformers embeddings for semantic retrieval over the chunked corpus, plus BM25 keyword retrieval (`rank-bm25` is in `requirements.txt`)
- **Citation-backed answers** — every answer carries book, chapter and page references (see "How citations work")
- **Provider-agnostic LLM gateway** — supports 5 LLM providers, swappable via a single config change, so the answering model is not hard-wired to one vendor
- **Tested** — 160 automated tests (pytest) across all 5 modules

## Architecture

The repository has 5 modules under `chatbot/`: `api`, `frontend`, `ingestion`, `llm_gateway` and `retriever`.

1. **Ingestion / indexing** — the 6 textbooks are split into 13,000+ chunks and embedded with Sentence-Transformers (`all-MiniLM-L6-v2`, 384-dim).
2. **Vector store** — chunks and embeddings are stored in ChromaDB (collection `waste_management_books`, persisted under `./data/chromadb`) for retrieval.
3. **Hybrid retrieval** — an incoming question is matched against the corpus through the ChromaDB + Sentence-Transformers retrieval pipeline, and the most relevant chunks are selected as context.
4. **LLM gateway** — the retrieved context and the question are passed to the configured LLM provider through a provider-agnostic gateway (5 providers supported, switched by a single config change).
5. **API + interface** — the service is exposed through a FastAPI backend, with a plain HTML/CSS/JavaScript front end served by the API itself.

## How citations work

Answers are generated from retrieved textbook chunks, not from the model's memory alone. Because each chunk originates from a known book, chapter and page range, the answer can be returned with that source attached: **book, chapter, and page**. That makes an answer verifiable — a reader can open the cited page and check whether the chatbot represented it correctly, instead of having to trust a confident-sounding paragraph.

## Tech stack

- **Language:** Python (>= 3.11, per `pyproject.toml`)
- **Vector store / retrieval:** ChromaDB, Sentence-Transformers, BM25
- **API:** FastAPI, Uvicorn
- **Front end:** Vanilla HTML / CSS / JavaScript
- **Tests:** pytest (160 tests, 5 modules)

## Running it

All commands below are taken from the repository itself (verified against the public repo on 2026-11-02). They have not been run end-to-end in this preparation — see the note at the end.

```bash
# 1. Clone the repository
git clone https://github.com/Sarosh548/Waste-management-chatbot.git
cd Waste-management-chatbot

# 2. Create and activate a virtual environment
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure the LLM provider
# Edit config.yaml at the repo root: set llm.provider and llm.model.
# Default: groq / llama-3.3-70b-versatile, temperature 0.1.
# Providers supported: groq, google, openai, anthropic, ollama.
# Put the matching key in your environment, e.g. GROQ_API_KEY,
# GOOGLE_API_KEY, OPENAI_API_KEY or ANTHROPIC_API_KEY — keys are
# loaded from the environment, not from config.yaml.

# 5. Build the textbook index
# The textbooks themselves are NOT in the repo, and the generated
# index (data/) is git-ignored, so a fresh clone must build it:
# place the PDFs in ./Books (the default in config.yaml), then run:
python -m chatbot.ingestion.cli --source ./Books
# To inspect an existing index instead: python -m chatbot.ingestion.cli --stats

# 6. Start the API (serves the frontend at the root URL)
uvicorn chatbot.api.main:app --reload
# API on port 8000 by default (config.yaml: server.port).
# Open http://localhost:8000 in a browser.
```

## Tests

160 automated tests, written with pytest, covering all 5 modules.

```bash
pytest
```

(`pyproject.toml` sets `testpaths = ["tests"]`; no extra test configuration is documented in the repo.)

## Status

Built in 2026 as a contribution project for a government-level waste management simulator. Stack: Python, ChromaDB, FastAPI, vanilla HTML/CSS/JS.

Author: Sarosh Qaiser — https://github.com/Sarosh548
