# SPEC.md: "The Human How"; Exact technical requirements, environment setup, implementation details, etc.

## Execution Description

Runs scripts/gpu-bench-rag-faiss-end2end.py: load rajpurkar/squad_v2, embed with BAAI/bge-small-en-v1.5, search FAISS IndexFlatIP, rerank with BAAI/bge-reranker-base, and generate with mistralai/Mistral-7B-v0.3 for the yaml query_count. output_format: csv Sweep dimensions: corpus_dataset, chunk_size, chunk_overlap, embedding_model, vector_db, llm_model, reranker_model, retrieval_strategy.

## Parameters

| Parameter | CLI Flag | Tested Values | Default | Description |
| --- | --- | --- | --- | --- |
| corpus_dataset | `--corpus-dataset` | smoke=rajpurkar/squad_v2, baseline=rajpurkar/squad_v2, extended=rajpurkar/squad_v2 | rajpurkar/squad_v2 | From Parameter list; see Execution Description With Parameters. |
| chunk_size | `--chunk-size` | smoke=64, baseline=256, extended=256 | 256 | From Parameter list; see Execution Description With Parameters. |
| chunk_overlap | `--chunk-overlap` | smoke=32, baseline=32, extended=32 | 32 | From Parameter list; see Execution Description With Parameters. |
| embedding_model | `--embedding-model` | smoke=BAAI/bge-small-en-v1.5, baseline=BAAI/bge-small-en-v1.5, extended=BAAI/bge-small-en-v1.5 | BAAI/bge-small-en-v1.5 | From Parameter list; see Execution Description With Parameters. |
| vector_db | `--vector-db` | smoke=faiss, baseline=faiss, extended=faiss | faiss | From Parameter list; see Execution Description With Parameters. |
| llm_model | `--llm-model` | smoke=mistralai/Mistral-7B-v0.3, baseline=mistralai/Mistral-7B-v0.3, extended=mistralai/Mistral-7B-v0.3 | mistralai/Mistral-7B-v0.3 | From Parameter list; see Execution Description With Parameters. |
| reranker_model | `--reranker-model` | smoke=BAAI/bge-reranker-base, baseline=BAAI/bge-reranker-base, extended=BAAI/bge-reranker-base | BAAI/bge-reranker-base | From Parameter list; see Execution Description With Parameters. |
| retrieval_strategy | `--retrieval-strategy` | smoke=dense, baseline=dense, extended=dense | dense | From Parameter list; see Execution Description With Parameters. |
| top_k | `--top-k` | smoke=5, baseline=5, extended=5 | 5 | From Parameter list; see Execution Description With Parameters. |
| query_length | `--query-length` | smoke=32, baseline=32, extended=32 | 32 | From Parameter list; see Execution Description With Parameters. |
| batch_size | `--batch-size` | smoke=2, baseline=8, extended=8 | 8 | From Parameter list; see Execution Description With Parameters. |
| query_count | `--query-count` | smoke=2, baseline=13, extended=40 | 13 | From Parameter list; see Execution Description With Parameters. |

## Invocation

```bash
Run scripts/gpu-bench-rag-faiss-end2end.py for squad_v2 retrieval, BGE embeddings, FAISS search, and Mistral generation
```

## Raw Output Format

raw_results.csv with pass1, pass2, and summary copies of the same aggregate, plus queries.csv with one row per measured query

check_name,end_to_end_query_latency_retrieval_generation_ms,generation_phase_time_to_first_token_ms,retrieval_latency_vector_search_ms,faiss_index_ntotal,generation_throughput_output_tokens_s,embedding_throughput_docs_s,index_backend,corpus_docs,chunk_count,query_count,chunk_overlap
summary,700,100,2.0,200,45,180,1,40,200,8,20

## Metrics

- **#1: E2E, ms** — stored as `end_to_end_query_latency_retrieval_generation_ms`.
- **#2: Generation TTFT, ms** — stored as `generation_phase_time_to_first_token_ms`.
- **#3: Retrieval latency** — stored as `retrieval_latency_vector_search_ms`.
- **#4: Gen throughput, tok/s** — stored as `generation_throughput_output_tokens_s`.
- **#5: Embedding throughput** — stored as `embedding_throughput_docs_s`.
- **#6: FAISS index vectors** — stored as `faiss_index_ntotal`.

## Framework

Runs scripts/gpu-bench-rag-faiss-end2end.py: load rajpurkar/squad_v2, embed with BAAI/bge-small-en-v1.5, search FAISS IndexFlatIP, rerank with BAAI/bge-reranker-base, and generate with mistralai/Mistral-7B-v0.3 for the yaml query_count. output_format: csv

## Installation and Execution Summary

Run scripts/gpu-bench-rag-faiss-end2end.py: load rajpurkar/squad_v2, embed with BAAI/bge-small-en-v1.5, search FAISS IndexFlatIP, rerank with BAAI/bge-reranker-base, and generate with mistralai/Mistral-7B-v0.3 for the yaml query_count, to measure end-to-end RAG latency and throughput

## Platform Portability

- **AMD (primary):** ```bash
Run scripts/gpu-bench-rag-faiss-end2end.py for squad_v2 retrieval, BGE embeddings, FAISS search, and Mistral generation
```
- **NVIDIA:** Native NVIDIA CUDA workload. Execute on the stated Ubuntu release with the host NVIDIA driver and CUDA userspace. ROCm porting notes do not apply.

## Model Context Protocols

- **Active:** None

## Execution-Loop Validation Contract

EXECUTION CHAIN: `run_benchmark.sh` ➔ raw output ➔ `scripts/parse_results.py` ➔ `results/benchmark.db` ➔ `scripts/validate_results.py`

This benchmark uses a lightweight, SQLite-integrated execution loop for result validation. All validation is performed by `scripts/validate_results.py`.

### Validation script usage

```bash
export BENCHMARK_PYTHON=/usr/bin/python3.13  # optional; select the installed interpreter

# After a live run:
".venv/bin/python" scripts/validate_results.py --db results/benchmark.db

# CI / no-GPU path (seeds fixture and validates it):
".venv/bin/python" scripts/validate_results.py --seed-fixture --quiet

# Override DB path via environment variable:
BENCHMARK_DB=tests/fixtures/benchmark.db \
  ".venv/bin/python" scripts/validate_results.py
```

### Run artifact contract

raw_results.csv with pass1, pass2, and summary copies of the same aggregate, plus queries.csv with one row per measured query

check_name,end_to_end_query_latency_retrieval_generation_ms,generation_phase_time_to_first_token_ms,retrieval_latency_vector_search_ms,faiss_index_ntotal,generation_throughput_output_tokens_s,embedding_throughput_docs_s,index_backend,corpus_docs,chunk_count,query_count,chunk_overlap
summary,700,100,2.0,200,45,180,1,40,200,8,20

```bash
bash run_benchmark.sh --help
bash run_benchmark.sh --profile smoke --validate
bash run_benchmark.sh --profile baseline --validate
bash run_benchmark.sh --profile extended --validate
```
`run_benchmark.sh --help` prints usage and exits. The harness calls `scripts/ensure_setup.sh` when `.setup_state` is absent.

### Required integrity checks (built into `validate_results.py`)

1. Latest run exists and `runs.status = 'ok'`.
2. `run.error_message` is NULL.
3. `started_at` and `finished_at` are valid ISO-8601 UTC strings.
4. All required aggregate metrics in `runs` are non-NULL and finite.
5. All required aggregate metrics are physically sensible (positive values). Runs scripts/gpu-bench-rag-faiss-end2end.py: load rajpurkar/squad_v2, embed with BAAI/bge-small-en-v1.5, search FAISS IndexFlatIP, rerank with BAAI/bge-reranker-base, and generate with mistralai/Mistral-7B-v0.3 for the yaml query_count. output_format: csv
6. At least 2 sample rows exist for the latest `run_id` (sweep coverage).
7. No sample has `status = 'error'`.
8. Runs scripts/gpu-bench-rag-faiss-end2end.py: load rajpurkar/squad_v2, embed with BAAI/bge-small-en-v1.5, search FAISS IndexFlatIP, rerank with BAAI/bge-reranker-base, and generate with mistralai/Mistral-7B-v0.3 for the yaml query_count. output_format: csv

### Baseline / Threshold configuration (`config/benchmark_config.yaml`)

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

Threshold key suffixes encode comparison direction when `thresholds:` is present: `_min` → observed value must be ≥ threshold. `_max` → observed value must be ≤ threshold. Informational `baselines:` ranges are not pass/fail gates.
