# PRD.md:  "The Why"; Product requirements, benchmark metadata table, high-level requirements, etc.

Product Requirements Document

"The Why"; Product requirements, benchmark metadata table, high-level requirements, etc. Defines the benchmark goal, validation objective, test name, benchmark number, category, and high-level success criteria.

## Benchmark Matrix Document Metadata (via benchmark_specification.json)

This PRD.md section is populated from benchmark_specification.json, which is the structured source of benchmark-specific product requirements.

## Workload Number
432

## Workload Name
RAG Pipeline Sweep

## Execution Summary (Run and Measure)
Run scripts/gpu-bench-rag-faiss-end2end.py: load rajpurkar/squad_v2, embed with BAAI/bge-small-en-v1.5, search FAISS IndexFlatIP, rerank with BAAI/bge-reranker-base, and generate with mistralai/Mistral-7B-v0.3 for the yaml query_count, to measure end-to-end RAG latency and throughput

## Main Goal
Measure end-to-end RAG retrieval plus generation performance

## Validation Objective
Validates squad_v2 retrieval plus BGE embedding, FAISS search, reranking, and Mistral generation for the yaml query count

## Workload Category
End-to-End Application Pipelines

## Validation Requirement

The benchmark must include an automated SQLite-integrated validation layer that verifies persisted results from `results/benchmark.db`. Validation must confirm:

1. The benchmark run completed successfully with no tool errors.
2. Required samples and aggregate metrics were persisted for every swept shape.
3. Metrics are finite and physically sensible (positive, within plausible bounds).
4. Measured values satisfy configured thresholds when the workload defines pass/fail gates.
5. The benchmark fails validation when required data is missing, invalid, or outside bounds.

## Non-Functional Requirements

| Requirement | Target |
|---|---|
| Automation | Runs to completion without manual intervention after `bash run_benchmark.sh` |
| Idempotency | Re-running `run_benchmark.sh` appends a new run; never corrupts existing rows |
| Persistence | All metrics survive script exit; `results/benchmark.db` is the durable record |
