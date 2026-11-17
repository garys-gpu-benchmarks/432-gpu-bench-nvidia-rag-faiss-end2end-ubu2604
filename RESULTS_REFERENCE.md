# Results: 432 RAG Pipeline Sweep

**Main goal:** Measure end-to-end RAG retrieval plus generation performance  
**Tools:** Run scripts/gpu-bench-rag-faiss-end2end.py for squad_v2 retrieval, BGE embeddings, FAISS search, and Mistral generation

Reference results measured by the maintainer on the system shown in each section (generator TEMPLATE_00_145 · recorded 2026-11-17). Your own runs write `results/RESULTS_ACTUAL.md` in the same format.

> Smoke runs use deliberately small sizes. They check that the workload installs and runs end to end; their numbers are not performance results. Baseline and extended are.

<!-- profile:smoke -->
## smoke

- **System:** 1× NVIDIA H100 NVL (95830 MiB) · AMD EPYC 9374F 32-Core Processor · Ubuntu 26.04.1 LTS · kernel 6.8.0-138-generic · driver 595.112.07
- **Run:** 2026-11-17T12:01:09Z · status **PASS** (exit 0)
- **Time (mm:ss):** setup 01:37 · benchmark 00:07 · whole run 01:54
- **Source:** run 432_20261117_120109
- **Command:** `bash run_benchmark.sh --smoke --chunk-size 64 --chunk-overlap 32 --corpus-dataset rajpurkar/squad_v2 --top-k 5 --embedding-model BAAI/bge-small-en-v1.5 --query-length 32 --batch-size 2 --llm-model mistralai/Mistral-7B-v0.3 --query-count 2 --reranker-model BAAI/bge-reranker-base`

| # | Metric | Result |
|---|---|---|
| 1 | E2E mean over queries, ms (end_to_end_query_latency_retrieval_generation_ms) | 670.5621460023394 |
| 2 | Generation TTFT, ms (generation_phase_time_to_first_token_ms) | 94.165084999986 |
| 3 | CPU FAISS retrieval latency, ms (retrieval_latency_vector_search_ms) | 0.09763200068846345 |
| 4 | Gen throughput, tok/s (generation_throughput_output_tokens_s) | 24.726854607393282 |
| 5 | Embedding throughput, chunks/s (embedding_throughput_chunks_s) | 585.9417981545649 |
| 6 | FAISS index vectors (faiss_index_ntotal) | 217.0 |
<!-- /profile:smoke -->
