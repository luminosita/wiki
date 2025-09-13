---
author: GPT-5
title: Building a Production-Ready Vector Database for RAG
date: 2025-09-06
tags:
  - vector-db
  - rag
  - chunking
categories:
  - AI
  - Documentation
version: 1.0
status: draft
description: Comprehensive plan for building a production-ready vector DB for internal documentation and RAG.
summary: |
  This document outlines a step-by-step plan for building a production-ready vector database 
  for internal company documentation to support Retrieval-Augmented Generation (RAG) in an AI-driven 
  Internal Developer Platform (IDP). It defines chunking strategies per documentation source, provides 
  authoring guidelines for creating chunk-friendly documentation, compares embedding models (with an emphasis 
  on smaller local models), and covers embedding optimization techniques. The document also reviews vector 
  database choices, metadata schema design, ingestion pipelines, retrieval orchestration, evaluation metrics, 
  and production-readiness considerations. A phased roadmap is included to guide implementation.
audience:
  - employees
  - managers
  - engineers
related_documents:
  - document-id-123
  - document-id-456
last_reviewed: 2025-09-06
next_review_due: 2026-03-06
confidentiality: internal
owner_department: Platform Engineering
---

# Building a Production-Ready Vector Database for RAG

## 1. Chunking Strategy (per source)
- **Wiki/Markdown:** split on headings, 200–600 tokens, 20% overlap.
- **Slack/Chat:** group by thread or 30-min window, 50–200 tokens, 20–40 token overlap.
- **Runbooks:** keep steps intact, 150–400 tokens, 30–80 overlap.
- **Ticketing:** split by description + comments, 100–400 tokens.
- **K8s/Terraform:** split per resource block, 50–300 tokens.
- **Policies/ADRs:** preserve sections, 300–800 tokens, 50–150 overlap.
- **Code/Comments:** keep docstrings/snippets intact, metadata-rich.
- **Troubleshooting/FAQ:** keep Q&A pairs intact, 150–500 tokens.

## 2. Authoring Guidelines
- Use clear headings and short paragraphs.
- Number steps in runbooks.
- Keep code in fenced blocks with summaries.
- Add structured frontmatter with version, tags, and ownership.
- Include summaries/TL;DR sections.
- Keep docs single-purpose.

## 3. Embedding Models
- **Local fast options:** `all-MiniLM-L6-v2`, `E5-base-v2`.
- **High-quality options:** BGE family, hosted services (OpenAI, Anthropic).
- **Recommendation:** start local (MiniLM/E5), evaluate, upgrade if needed.

## 4. Embedding Optimizations
- Quantization (ONNX, int8).
- Batching & async inference.
- Hybrid embeddings (fast coarse + rerank).
- Periodic re-embedding with hashing.
- Cache frequent queries.

## 5. Vector DB Options
- **OSS:** Qdrant, Milvus, Weaviate, pgvector.
- **Managed:** Pinecone.
- **Recommendation:** Qdrant/Milvus self-hosted for internal use.

## 6. Metadata Schema
Suggested fields: `doc_id`, `source`, `path`, `title`, `chunk_index`, `team`, `sensitivity`, `version`, `timestamps`, `tags`, `hash`.

## 7. Ingestion Pipeline
1. Extract from sources.
2. Normalize & clean.
3. Chunk using source-specific rules.
4. Deduplicate via hash.
5. Embed in batches.
6. Upsert into vector DB.
7. Monitor & re-index on change.

## 8. Retrieval & RAG
- Two-stage: ANN retrieval → reranking.
- Include provenance in prompts.
- Route queries to specialized agents.

## 9. Evaluation
- Use Recall@k, MRR, latency, cost.
- Build ground-truth QA dataset.
- Run A/B tests across models & chunk strategies.

## 10. Security & Production
- Encrypt at rest & in transit.
- Role-based access.
- PII redaction.
- Logging & monitoring.
- Backup & disaster recovery.

## 11. Roadmap
- **Phase 0:** Source inventory & dataset.
- **Phase 1:** PoC with 2–3 sources + Qdrant/MiniLM.
- **Phase 2:** Optimize embeddings, add reranker.
- **Phase 3:** Scale, integrate, secure, monitor.

---

## Chat

You asked:
----------

I want to create a comprehensive vector DB for internal company documentation. Vector DB should be used in conjunction with RAG, and together with some custom-built MCP servers and publicly available MCP servers form a powerful software engineering IDP platform, solely driven by AI queries instead of a standard UI approach (i.e. Backstage, Port, etc). The idea is to ingest various documentation sources (knowledge base/wiki markdown files, Slack conversations, runbooks, ticketing system, Kubernetes manifests, Terraform configurations, security policies, architecture decisions, troubleshooting guides, code comments, example snippets, etc) into a vector DB to be used as a proper context when communicating with AI models through AI Agents. As a first step, I would like to establish the best chunking strategy per documentation source and establish clear guidelines on how documentation should be created from scratch to enable the best chunking results. Further on, I would need to research various AI models for vector DB embeddings. Ideally, a smaller local AI model should be used for such a procedure. Next step should be to apply embedding optimizations. The final product should be production-ready vector DB.

---

ChatGPT Replied:
----------------

Thought for 15s

Nice — this is a great project and you’ve already described the scope clearly. Below I’ve put together a practical, production-ready plan you can act on immediately: concrete chunking strategies per source, authoring guidelines to make future docs “chunk-friendly”, recommended embedding models (including small/local options), embedding-optimization techniques, a comparison of vector DB choices, the ingestion/ops architecture, evaluation metrics, and a rollout roadmap with checkpoints.

I used recent industry guidance and benchmarks while preparing these recommendations. I’ve cited the most important references inline so you can validate or dive deeper. If you want, I can immediately generate sample ingestion code (Python + LangChain-style pipeline), chunker configs, and an example metadata schema you can import into your CI/CD pipeline.

Executive summary
=================

*   **Goal:** ingest heterogeneous internal docs into a vector DB to serve as context for RAG-driven AI Agents in a software-engineering IDP (internal developer platform).
    
*   **High-level approach:** standard ETL pipeline: _extract → clean/normalize → chunk (smart split) → embed → upsert (vector DB) → monitor/re-embed on change_.
    
*   **Key choices:** use semantic chunking (semantic boundaries + token limits) + metadata-rich indexing (source, path, object type, K/V tags, version, hash) → prefer open-source self-hosted vector DBs for internal docs (Qdrant / Milvus / Weaviate / pgvector) or managed Pinecone for SaaS. [Git for Data - lakeFS](https://lakefs.io/blog/12-vector-databases-2023/?utm_source=chatgpt.com)[DataCamp](https://www.datacamp.com/blog/the-top-5-vector-databases?utm_source=chatgpt.com)
    

* * *

1) Chunking strategy — per source (concrete rules)
==================================================

General rules applied everywhere:

*   Prefer **semantic splits** (paragraphs, headings, code blocks) over blind fixed-character cuts.
    
*   Target chunk sizes in **tokens** (not characters) because embeddings and LLM prompts are token-based. For practical conversion, 1 token ≈ 4 characters (English).
    
*   **Chunk overlap** 10–25% to preserve context across chunk boundaries (higher for runbooks/troubleshooting).
    
*   Preserve explicit boundaries: keep code blocks, YAML/JSON documents, and manifests intact wherever possible.
    
*   Store (as metadata) the **source path**, **document title/heading**, **line/char offsets**, **version or commit hash**, and **chunk\_id**.
    

Concrete per-source defaults (start here; tune per domain with evaluation):

1.  Knowledge base / wiki (Markdown)
    
    *   Split on headings (H1–H3) first. If heading content > 600 tokens, sub-split by paragraph or sentence.
        
    *   Target chunk size: **200–600 tokens** (≈ 800–2,400 chars). Overlap **50–100 tokens** (≈ 20%).
        
    *   Keep intra-document links & heading metadata in chunk metadata.
        
2.  Slack / chat transcripts
    
    *   Preprocess: remove ephemeral noise (timestamps if duplicated), normalize usernames.
        
    *   Group messages by **thread** or by sliding time window (e.g., thread or 30-minute window).
        
    *   Target chunk size: **50–200 tokens** (shorter because conversational turns are small). Overlap **20–40 tokens**.
        
3.  Runbooks / playbooks / run commands
    
    *   Keep step-by-step numbered steps together where possible.
        
    *   Target chunk: **150–400 tokens**; set overlap **30–80 tokens** so a step and its context aren’t split.
        
    *   Mark **severity/impact** and **preconditions** in metadata.
        
4.  Ticketing (Jira/ServiceNow)
    
    *   Split by comment thread + description. Keep attachments referenced (store link + optionally separate vector).
        
    *   Target chunk: **100–400 tokens**. Include status, assignee, dates in metadata.
        
5.  Kubernetes manifests / Terraform / config files
    
    *   Treat each resource block / module as one chunk where possible (e.g., a K8s manifest YAML resource).
        
    *   For long files, split per resource and by logical sections (metadata/spec).
        
    *   Target chunk size: **50–300 tokens** depending on resource. Preserve raw snippet in metadata/attachment.
        
6.  Security policies / compliance docs / architecture decisions (ADRs)
    
    *   Preserve whole policy sections. Target **300–800 tokens**. Overlap **50–150**.
        
    *   Ensure tags for **sensitivity classification** and **retention window**.
        
7.  Code comments and example snippets
    
    *   Extract comment blocks and inline docstrings, and treat them as separate chunks.
        
    *   For code examples: keep the whole code snippet as one chunk (or file-level chunks).
        
    *   Add metadata: language, filename, repo + commit.
        
8.  Troubleshooting guides / FAQs
    
    *   Keep question+answer pairs together. Target **150–500 tokens** with high overlap to ensure context.
        

References on chunking best practices: Unstructured.io and Databricks + several primers — follow semantic splitting and token limits. [Unstructured](https://unstructured.io/blog/chunking-for-rag-best-practices?utm_source=chatgpt.com)[Databricks Community](https://community.databricks.com/t5/technical-blog/the-ultimate-guide-to-chunking-strategies-for-rag-applications/ba-p/113089?utm_source=chatgpt.com)

* * *

2) Authoring guidelines to maximize chunking quality (how docs should be written)
=================================================================================

Make docs _chunk-friendly_ going forward:

*   Use clear headings (H1/H2/H3) and short paragraphs (1–4 sentences). Headings enable meaningful semantic splits.
    
*   For runbooks: number steps and keep steps under ~100–150 tokens where possible.
    
*   Keep code snippets and manifests as fenced blocks and include a one-line summary comment above them.
    
*   Tag documents with structured front-matter (YAML/JSON) when possible, e.g.:
    
        title: "K8s: Deploy service X"
        team: "platform"
        type: "runbook"
        sensitivity: "internal"
        version: "v1.2"
        last_updated: "2025-08-01"
        
    
*   Include a short “summary” or TL;DR at top (1–3 sentences) — this produces excellent small chunks for retrieval.
    
*   Prefer single-responsibility documents (one runbook = one incident type) — avoids large monolithic files that are hard to chunk well.
    

* * *

3) Embedding model choices — local & production options
=======================================================

**Tradeoffs**: quality vs cost/latency vs privacy/control.

**Small / local / fast (good starting point)**

*   **sentence-transformers/all-MiniLM-L6-v2** — compact (384-dim), fast on CPU/GPU, good baseline for paragraph embeddings. Great for self-hosted inference at scale. [Hugging Face](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2?utm_source=chatgpt.com)[Medium](https://datastax.medium.com/how-to-create-vector-embeddings-in-python-8be63d4cef9f?utm_source=chatgpt.com)
    
*   **E5 / intfloat/e5-base-v2** — strong open-source model for embeddings (better quality in many benchmarks than earlier MiniLM variants). Consider if you want improved quality at a modest cost. [supermemory](https://supermemory.ai/blog/best-open-source-embedding-models-benchmarked-and-ranked/?utm_source=chatgpt.com)
    

**Higher quality / state-of-the-art (larger, possibly hosted):**

*   **bge-en / BGE family** — top performance on many embedding leaderboards, but heavier. Good when retrieval quality is paramount. [supermemory](https://supermemory.ai/blog/best-open-source-embedding-models-benchmarked-and-ranked/?utm_source=chatgpt.com)
    
*   **OpenAI / Anthropic / Sagemaker hosted embeddings** — often offer state-of-the-art embeddings with managed infra (if you are okay with SaaS). (If you require full internal-only, pick local open-source models.)
    

**Specialized / SOTA experiments**

*   AnglE / Llama-based embedding variants for certain datasets have strong performance; consider them for specialized similarity tasks. [Hugging Face Forums](https://discuss.huggingface.co/t/how-to-calculate-embeddings-with-llama-2-model/57026?utm_source=chatgpt.com)
    

**Recommendation for your use-case**

*   Start with **all-MiniLM-L6-v2** or **E5** self-hosted for ingestion (fast, low GPU needs), run evaluation (RR@k, MRR) and then upgrade to BGE or a managed high-quality service only if retrieval quality is insufficient. [Hugging Face](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2?utm_source=chatgpt.com)[supermemory](https://supermemory.ai/blog/best-open-source-embedding-models-benchmarked-and-ranked/?utm_source=chatgpt.com)
    

* * *

4) Embedding optimizations (practical techniques)
=================================================

1.  **Quantization & ONNX** — convert models to ONNX and apply int8/4 quantization to accelerate inference or reduce memory. ONNX Runtime provides standard recipes. Quantizing can yield 2–5× speedups with acceptable quality tradeoffs. [ONNX Runtime](https://onnxruntime.ai/docs/performance/model-optimizations/quantization.html?utm_source=chatgpt.com)[Medium](https://medium.com/nixiesearch/how-to-compute-llm-embeddings-3x-faster-with-model-quantization-25523d9b4ce5?utm_source=chatgpt.com)
    
2.  **Batching & async pipelines** — batch small chunks to GPUs to increase throughput.
    
3.  **Dimensionality tuning / projection** — consider PCA or light re-projection only if storage/latency is a constraint (beware of quality loss).
    
4.  **Hybrid embedding strategy** — produce two embeddings per chunk: (fast small) for coarse retrieval + (higher-quality) for reranking. This reduces cost while preserving precision.
    
5.  **Cache popular queries & warm embeddings** — for internal tools, many queries are repeated; cache their retrieval results.
    
6.  **Periodic re-embedding** — re-embed only changed docs (use content hash) to avoid reprocessing everything.
    
7.  **Quantized indexing in DB** — some DBs support on-disk quantized indices to reduce RAM. (See DB docs.)
    

Benchmarks and guides show ONNX quantization + transformer optimizations dramatically speed embedding pipelines while being production practical. [Medium](https://medium.com/nixiesearch/how-to-compute-llm-embeddings-3x-faster-with-model-quantization-25523d9b4ce5?utm_source=chatgpt.com)[AI Accelerator Institute](https://www.aiacceleratorinstitute.com/how-to-8-bit-quantize-large-models-using-bits-and-bytes/?utm_source=chatgpt.com)

* * *

5) Vector DB choices — pros/cons & recommendation
=================================================

Short list and quick guidance (sources linked):

*   **Pinecone (managed)** — easiest SaaS experience, fully managed, simple API, good for rapid production if SaaS is acceptable. [Git for Data - lakeFS](https://lakefs.io/blog/12-vector-databases-2023/?utm_source=chatgpt.com)
    
*   **Qdrant (OSS & managed)** — excellent open-source, lightweight, good full-text + vector, good for internal self-hosted deployments. [Medium](https://medium.com/tech-ai-made-easy/vector-database-comparison-pinecone-vs-weaviate-vs-qdrant-vs-faiss-vs-milvus-vs-chroma-2025-15bf152f891d?utm_source=chatgpt.com)[Medium](https://mehmetozkaya.medium.com/exploring-vector-databases-pinecone-chroma-weaviate-qdrant-milvus-pgvector-and-redis-f0618fe9e92d?utm_source=chatgpt.com)
    
*   **Milvus (OSS)** — scale and performance for very large vector stores; robust clustering & GPU support. [DataCamp](https://www.datacamp.com/blog/the-top-5-vector-databases?utm_source=chatgpt.com)
    
*   **Weaviate** — semantic modules, schema-driven, built-in vectorization options & GraphQL; useful if you want an integrated semantic layer. [Medium](https://medium.com/tech-ai-made-easy/vector-database-comparison-pinecone-vs-weaviate-vs-qdrant-vs-faiss-vs-milvus-vs-chroma-2025-15bf152f891d?utm_source=chatgpt.com)
    
*   **Chroma / Faiss / pgvector** — good options depending on maturity, footprint, and data model needs. Faiss is library-level (you’ll manage infra); pgvector is excellent for small-scale and SQL integration. [Medium](https://medium.com/tech-ai-made-easy/vector-database-comparison-pinecone-vs-weaviate-vs-qdrant-vs-faiss-vs-milvus-vs-chroma-2025-15bf152f891d?utm_source=chatgpt.com)[GeeksforGeeks](https://www.geeksforgeeks.org/dbms/top-vector-databases/?utm_source=chatgpt.com)
    

**Recommended approach for your IDP**:

*   If **strictly internal + want control**: start with **Qdrant** or **Milvus** (self-hosted in a k8s cluster with autoscaling GPU/CPU workers for embedding ops). Qdrant is developer-friendly and light; Milvus is great when scale and GPU index building are needed. [DataCamp](https://www.datacamp.com/blog/the-top-5-vector-databases?utm_source=chatgpt.com)[Medium](https://medium.com/tech-ai-made-easy/vector-database-comparison-pinecone-vs-weaviate-vs-qdrant-vs-faiss-vs-milvus-vs-chroma-2025-15bf152f891d?utm_source=chatgpt.com)
    
*   If **managed/SaaS is acceptable**: consider **Pinecone** for fast time-to-market. [Git for Data - lakeFS](https://lakefs.io/blog/12-vector-databases-2023/?utm_source=chatgpt.com)
    

* * *

6) Metadata schema (suggested fields — store alongside each vector)
===================================================================

*   `doc_id` (unique)
    
*   `source` (wiki, slack, runbook, repo, jira, etc.)
    
*   `path` or `url` (link to original)
    
*   `title` / `heading`
    
*   `chunk_index`
    
*   `char_start` / `char_end`
    
*   `token_count`
    
*   `team` / `owner`
    
*   `sensitivity` / `classification` (internal/confidential)
    
*   `version` (commit hash or doc version)
    
*   `created_at` / `last_updated`
    
*   `embedding_model` + `embedding_dim`
    
*   `language`
    
*   `tags` (k/v pairs e.g., service:payments, component:k8s)
    
*   `sha256` (content hash for change detection)
    

Keeping these fields consistent enables precise filtering at retrieval time (e.g., “search only docs owned by team X and not labeled confidential”).

* * *

7) Ingestion pipeline (detailed, actionable)
============================================

1.  **Extractor**: for each source, use dedicated connectors:
    
    *   Git repos: read latest files + commit info (use git perms).
        
    *   Slack: export via API + threads.
        
    *   Jira/Ticketing: API pulls (use incremental pulls).
        
    *   K8s/Terraform: extract from repos or runtime clusters.
        
2.  **Normalizer**: remove noise, strip PII if required, normalize whitespace, extract front-matter.
    
3.  **Chunker**: apply per-source rules described above; emit chunks + metadata.
    
4.  **Deduper & canonicalizer**: hash chunks, dedupe identical content, mark duplicates.
    
5.  **Embedders**: batch chunks and compute embeddings (use quantized model for speed).
    
6.  **Upsert to vector DB**: use upsert semantics by `doc_id`+`chunk_index`. Use namespaces/collections per environment/team if beneficial.
    
7.  **Reranker (optional)**: for high precision, run a reranker (higher-quality embed or cross-encoder) on top-k results.
    
8.  **Monitor**: logging, metrics (QPS, latency, fail rates), and alerting for pipeline failures.
    
9.  **Re-indexing triggers**: when source changes (webhook/commit), re-chunk & re-embed only affected docs.
    

* * *

8) Retrieval & RAG orchestration
================================

*   Use a two-stage retrieval:
    
    1.  **ANN Retrieval**: fetch top N (e.g., 50) by vector similarity with light filters (metadata).
        
    2.  **Rerank**: use a semantic reranker or cross-encoder to refine top K (e.g., reduce to 5–10).
        
*   **Prompt engineering**: include chunk provenance and highlight “do not hallucinate” instruction. Provide source citations for answers (chunk `path` + `chunk_index`).
    
*   **Agent orchestration**: route queries to specialized agents (security, infra, development) depending on intent classifier (use a small intent classifier model on the query).
    

* * *

9) Evaluation & metrics (how to validate quality)
=================================================

Create a testbench with:

*   **Ground-truth QA pairs** (internal subject matter experts create ~500–2,000 pairs).
    
*   Metrics:
    
    *   **Recall@k** (was the correct doc among top-k retrieved?), target ≥ 90% for critical runbooks.
        
    *   **MRR / MAP** for ranking quality.
        
    *   **Latency (ms)** for retrieval + reranking.
        
    *   **Token cost** for LLM calls in RAG.
        
    *   **Answer fidelity** via human review (accuracy, hallucination rate).
        
*   Run A/B testing between embedding models (MiniLM vs E5 vs BGE) and chunk sizes.
    

* * *

10) Security, access control, compliance & production readiness
===============================================================

*   **Encryption in transit & at rest** for vectors and metadata.
    
*   **Role-based access control** to vector DB namespaces/collections.
    
*   **Audit logs** for queries and re-index operations.
    
*   **PII redaction** policy during extraction (or flagging for restricted access).
    
*   **Backups & snapshots** of indexes and metadata.
    
*   **Rate limiting** for agent queries to avoid data exposure.
    
*   **SLA & recovery**: plan for index rebuild timelines, warmup strategies.
    

* * *

11) Roadmap & deliverables (practical phased plan)
==================================================

Phase 0 — Discovery (1–2 sprints)

*   Inventory sources, estimate volumes, classify sensitivity.
    
*   Build test dataset for evaluation.
    

Phase 1 — PoC (2–4 sprints)

*   Implement connectors for 2–3 source types (e.g., wiki, git repos, slack).
    
*   Use **all-MiniLM-L6-v2** or **E5** self-hosted + Qdrant/Milvus. Run evaluation.
    

Phase 2 — Harden & Optimize (2–4 sprints)

*   Add quantization (ONNX), batching, reranker.
    
*   Add metadata filtering and security controls.
    

Phase 3 — Scale & Integrate (ongoing)

*   Add additional sources, autoscaling, monitoring, governance, CI for reindexing.
    
*   Add managed/hosted options for production if needed.
    

* * *

12) Quick actionable artefacts I can produce right now (tell me which you want and I’ll generate)
=================================================================================================

*   Sample Python ingestion pipeline (connectors + chunker + embed + upsert skeleton) ready-to-run.
    
*   Chunker config (YAML) with per-source rules you can drop into your pipeline.
    
*   Metadata schema as JSON/SQL table DDL.
    
*   Evaluation runner that calculates Recall@k and MRR given ground-truth pairs.
    
*   Cost/infra estimate for 100k–10M chunks (rough sizing).
    

* * *

Key references I used (most load-bearing)
=========================================

*   Vector DB landscape & managed vs OSS, quick comparisons. [Git for Data - lakeFS](https://lakefs.io/blog/12-vector-databases-2023/?utm_source=chatgpt.com)[Medium](https://medium.com/tech-ai-made-easy/vector-database-comparison-pinecone-vs-weaviate-vs-qdrant-vs-faiss-vs-milvus-vs-chroma-2025-15bf152f891d?utm_source=chatgpt.com)
    
*   Chunking best practices and strategies for RAG. [Unstructured](https://unstructured.io/blog/chunking-for-rag-best-practices?utm_source=chatgpt.com)[Databricks Community](https://community.databricks.com/t5/technical-blog/the-ultimate-guide-to-chunking-strategies-for-rag-applications/ba-p/113089?utm_source=chatgpt.com)
    
*   MiniLM / sentence-transformers model page (practical local option). [Hugging Face](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2?utm_source=chatgpt.com)
    
*   Benchmarks and rankings for open-source embedding models (E5, BGE). [supermemory](https://supermemory.ai/blog/best-open-source-embedding-models-benchmarked-and-ranked/?utm_source=chatgpt.com)
    
*   ONNX quantization and transformer quantization guides for optimization. [ONNX Runtime](https://onnxruntime.ai/docs/performance/model-optimizations/quantization.html?utm_source=chatgpt.com)[Medium](https://medium.com/nixiesearch/how-to-compute-llm-embeddings-3x-faster-with-model-quantization-25523d9b4ce5?utm_source=chatgpt.com)
    
