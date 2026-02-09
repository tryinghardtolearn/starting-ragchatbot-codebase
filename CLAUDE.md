# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A RAG (Retrieval-Augmented Generation) chatbot that answers questions about course materials. Python/FastAPI backend with vanilla JS frontend. Uses ChromaDB for vector storage and Anthropic Claude for AI responses.

## Commands

```bash
# Install dependencies
uv sync

# Run the app (from repo root)
./run.sh
# Or manually:
cd backend && uv run uvicorn app:app --reload --port 8000

# Access points
# Web UI: http://localhost:8000
# API docs: http://localhost:8000/docs
```

No test or lint tooling is configured.

## Architecture

**Backend (`backend/`)** — FastAPI application:

- `app.py` — FastAPI server, serves frontend static files, exposes `/api/query` (POST) and `/api/courses` (GET)
- `rag_system.py` — Main orchestrator that coordinates document processing, vector search, and AI generation
- `document_processor.py` — Parses course text files from `docs/`, extracts metadata and lessons, chunks text by sentences with configurable overlap
- `vector_store.py` — ChromaDB interface with two collections: `course_catalog` (metadata) and `course_content` (chunks). Uses SentenceTransformer (`all-MiniLM-L6-v2`) for embeddings
- `ai_generator.py` — Claude API integration using tool-use pattern; Claude decides when to search course content vs answer directly
- `search_tools.py` — Defines `CourseSearchTool` for Claude's tool-use, handles search execution and source formatting
- `session_manager.py` — Conversation history tracking per session ID
- `config.py` — Central configuration (model names, chunk sizes, search limits) loaded from env vars
- `models.py` — Pydantic models: `Course`, `Lesson`, `CourseChunk`

**Frontend (`frontend/`)** — Vanilla HTML/JS/CSS chat interface with course stats sidebar and suggested questions.

**Documents (`docs/`)** — Course material text files with structured format (title, link, instructor, lessons). Auto-loaded on app startup.

## Key Design Patterns

- **Tool-use RAG**: Rather than always retrieving, the AI decides via Claude tool-use when to search the vector store
- **Singleton RAG system**: Initialized once at app startup, shared across requests
- **Persistent ChromaDB**: Vector data in `backend/chroma_db/` survives restarts
- **Sentence-based chunking**: Documents split by sentences with overlap (800 char chunks, 100 char overlap)

## Configuration

Key settings in `config.py`:
- `ANTHROPIC_MODEL`: claude-sonnet-4-20250514
- `EMBEDDING_MODEL`: all-MiniLM-L6-v2
- `CHUNK_SIZE`: 800, `CHUNK_OVERLAP`: 100
- `MAX_RESULTS`: 5 search results
- Requires `ANTHROPIC_API_KEY` in `backend/.env`

## Dependencies

Python 3.13+, managed with `uv`. Key packages: `fastapi`, `uvicorn`, `anthropic`, `chromadb`, `sentence-transformers`, `python-dotenv`.
