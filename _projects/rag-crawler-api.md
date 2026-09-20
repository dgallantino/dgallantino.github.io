---
title: RAG crawler API
summary: A hobby RAG pipeline with a real HTTP API — chunk, embed, retrieve, optionally answer — while the crawler is still a skeleton.
stack:
  - Python
  - FastAPI
  - PostgreSQL
repo: https://github.com/dgallantino/rag-crawler-api
featured: true
status: shipped
date: 2026-09-07
---

I wanted a RAG stack I could poke at without a giant vendor demo. Markdown goes in, vectors land in PostgreSQL with pgvector, and you can retrieve chunks or ask for an answer with citations. FastAPI handlers stay thin; Celery does the embedding work; OpenRouter supplies embeddings, completions, and rerank.

The `/v1` routes are wired: collections, document upload, retrieve, and query, authenticated with an API key. Health checks exist. The crawler library (Playwright + BeautifulSoup) is still skeletal and not connected to the API or the database — the README is honest about that.

This is a learning project, not a hosted product. Rate limits and Alembic migrations are still on the roadmap.
