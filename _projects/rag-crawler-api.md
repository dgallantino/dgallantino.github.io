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

Markdown goes in through the CLI or `POST /v1/documents`. A Celery worker chunks it, embeds it, and stores the vectors on `document_chunks` in PostgreSQL with pgvector. `POST /v1/retrieve` returns ranked chunks. `POST /v1/query` returns an answer that cites those chunks.

FastAPI handlers stay thin and call the same services as `python -m app.cli`. The API key is the tenant. OpenRouter supplies the embedding, the optional rerank, and the completion. Docker Compose runs the API, Postgres, Redis, and the worker.

The crawler under `app/crawler/` (Playwright and BeautifulSoup) is a library only. `run-crawler` raises `NotImplementedError`. the crawler part still mostly skeletal code.


## How it is built

At PT Teknologi Integrasi Optima I worked on an AI customer-service chat on WhatsApp. People could book and get the information they needed for that booking in the same conversation. I liked that shape, and I wanted to learn the retrieval side of it properly. This repo is that study: a small model, with a path I can explain, answering accurately enough to be worth calling an assistant.

The models are deliberately ordinary. Embeddings default to `openai/text-embedding-3-small` (1536 dimensions). Answers default to `openai/gpt-4o-mini`. Rerank is `cohere/rerank-v3.5` and off unless you ask for it. I did not want a framework that hides the pipeline. `langchain-text-splitters` only does the first split (`RecursiveCharacterTextSplitter`, tiktoken `cl100k_base`). After that the code is mine: merge, metadata, hybrid search, and the prompt.

Chunking aims at markdown sections, not a flat character window. The splitter tries `\n## `, `\n### `, `\n#### `, then blank lines, then words. Defaults are 500 tokens max, a 10% overlap (50 tokens), and a 300-token floor: consecutive short pieces are joined until they would pass the max. Each chunk keeps the nearest heading as `section_header`, any headings inside the text as `all_headings`, and YAML frontmatter `tags` on every chunk from that file. The vector is not the raw body. `build_contextualized_embedding_text` prefixes `Title:` and `Heading:` before the embed call, in batches of 100. The row’s `content`, the full-text column, the rerank input, and the text the model sees stay the raw chunk. Query embeddings stay unprefixed, so the query is not pretending to be a section.

I stored the vectors in Postgres instead of a purpose-built vector database. `chunk_vector` is pgvector `Vector(1536)` with an HNSW index (`vector_cosine_ops`, `m=16`, `ef_construction=64`). Search uses cosine distance and turns it into a score with `clamp(1 - distance, 0, 1)`. Around that query the session sets `hnsw.ef_search` to at least `top_k * 2` (floor 40), `hnsw.iterative_scan` to `relaxed_order`, and `hnsw.max_scan_tuples` to 20000. Lexical search is a generated `content_tsv` (`to_tsvector('simple', …)`) with a GIN index. `simple` does not stem, which matters because the text I care about is Indonesian as well as English. A GIN index on `chunk_metadata` supports containment filters.

Retrieval runs both legs. Dense search and Postgres full-text (`websearch_to_tsquery` on `simple`, rank with `ts_rank_cd`) each pull `top_k * 2`, then Reciprocal Rank Fusion merges them with `k=60`. A chunk that only one leg found still gets that leg’s score. `HYBRID_SEARCH_ENABLED=false` drops back to vectors only. If rerank is on, the pool is `top_k * 4` before Cohere; any rerank failure keeps the fused order. Tenant scope never sits in the SQL filters. `retrieval_service` resolves the caller’s collections and passes UUIDs in.

Before the completion call, adjacent chunks from the same document are merged when `chunk_index` differs by one. Overlap that is already a suffix of the previous chunk is stripped, and the group is scored as the max of its members. The system prompt is one rule: answer from the context, and say when the context is not enough. Sources on the response are still the original chunks, with `chunk_id`, `document_id`, `chunk_index`, and score.

Ingest is a worker, not the request. Upload writes the `Document` row and Celery runs `process_document`: status `processing`, then chunk, embed, store. Before the write it row-locks the document and drops the result if `content` or `title` changed underneath it. Chunks for that document are deleted and inserted again. Status events are `chunking`, `embedding`, and `storing`, then `success` or `failed`.

On the markdown I used for testing, the first retrieved chunk was usually the one that answered the question, and that made the small model much easier to trust. That corpus is not production traffic, and I have not stress-tested it. Pytest covers chunking, hybrid search, the vector index, and the HTTP routes, and live OpenRouter calls are an opt-in e2e. There is still no golden question set and no human scoring pass, which is the check I want before I treat the accuracy as a number. Rate limiting is a column on `SystemUser` and is not enforced. Schema changes are still `create_all` in development, not Alembic. The API, the worker, auth, and Compose are the shape I want for real use. The crawler, the eval, and those operational gaps are what is still missing.
