# RAG Pipeline Sweep Benchmark

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![CI](https://github.com/garys-gpu-benchmarks/432-gpu-bench-nvidia-rag-faiss-end2end-ubu2604/actions/workflows/ci.yml/badge.svg)](https://github.com/garys-gpu-benchmarks/432-gpu-bench-nvidia-rag-faiss-end2end-ubu2604/actions/workflows/ci.yml)

Measures end-to-end RAG retrieval plus generation performance. Workload 432 of the NVIDIA Ubuntu 26.04 bundle.

## Quick Start

On an Ubuntu 26.04 host, as root or with sudo:

```bash
git clone https://github.com/garys-gpu-benchmarks/432-gpu-bench-nvidia-rag-faiss-end2end-ubu2604.git
cd 432-gpu-bench-nvidia-rag-faiss-end2end-ubu2604
sudo bash setup.sh --assume-yes
bash run_benchmark.sh --smoke
```

`run_benchmark.sh` runs setup by itself if it has not been run yet. Each run prints its metrics when it finishes.

The model `mistralai/Mistral-7B-v0.3` is gated on Hugging Face. Set `HF_TOKEN` before setup (see [Troubleshooting](#troubleshooting)).

## Profiles and Parameters

| Parameter | Flag | smoke | baseline | extended |
|---|---|---|---|---|
| `chunk_size` | `--chunk-size` | 64 | 256 | 256 |
| `chunk_overlap` | `--chunk-overlap` | 32 | 32 | 32 |
| `corpus_dataset` | `--corpus-dataset` | rajpurkar/squad_v2 | rajpurkar/squad_v2 | rajpurkar/squad_v2 |
| `top_k` | `--top-k` | 5 | 5 | 5 |
| `embedding_model` | `--embedding-model` | BAAI/bge-small-en-v1.5 | BAAI/bge-small-en-v1.5 | BAAI/bge-small-en-v1.5 |
| `query_length` | `--query-length` | 32 | 32 | 32 |
| `batch_size` | `--batch-size` | 2 | 8 | 8 |
| `llm_model` | `--llm-model` | mistralai/Mistral-7B-v0.3 | mistralai/Mistral-7B-v0.3 | mistralai/Mistral-7B-v0.3 |
| `query_count` | `--query-count` | 2 | 45 | 140 |
| `reranker_model` | `--reranker-model` | BAAI/bge-reranker-base | BAAI/bge-reranker-base | BAAI/bge-reranker-base |

Pick a profile with `--smoke` (default), `--baseline` or `--extended`. Any flag in the table overrides that profile's value:

```bash
bash run_benchmark.sh --baseline --batch-size 4
bash run_benchmark.sh --help        # every option
```

Profile values live in `config/benchmark_config.yaml`.

## Metrics

| # | Metric | Key |
|---|---|---|
| 1 | E2E mean over queries, ms | `end_to_end_query_latency_retrieval_generation_ms` |
| 2 | Generation TTFT, ms | `generation_phase_time_to_first_token_ms` |
| 3 | CPU FAISS retrieval latency, ms | `retrieval_latency_vector_search_ms` |
| 4 | Gen throughput, tok/s | `generation_throughput_output_tokens_s` |
| 5 | Embedding throughput, chunks/s | `embedding_throughput_chunks_s` |
| 6 | FAISS index vectors | `faiss_index_ntotal` |

### Notes

- The smoke profile is a quick check with reduced sizes and iteration counts. Use `--baseline` or `--extended` for representative numbers.
- Real HF models and squad_v2 download on first run.
- Smoke query_count is 2.
- FAISS build and Mistral generate are HBM- and time-intensive.

## Output

| Path | Contents |
|---|---|
| `results/raw/YYYYMMDD_HHMMSSZ_WORKLOAD-<NNN>_SYSTEM-<hostname>/` | Raw tool output, `metrics_summary.txt`, and `hardware_info.txt`, `software_info.txt`, `errors_info.txt` |
| `results/benchmark.db` | SQLite history of every run |
| `results/summary.json` | Metrics from the most recent run |
| `/var/opt/benchmarks/runtime_ledger.csv` | One row per run, shared by every workload on the host |

Every run is checked for completeness and against any `thresholds:` or `baselines:` in `config/benchmark_config.yaml`. A run that fails the checks exits non-zero.

## Requirements

- Ubuntu 26.04, root or sudo for `setup.sh`
- NVIDIA GPU with CUDA 13.3 (validated on H100 SXM 80GB)
- CUDA Runtime, PyTorch-CUDA, Hugging Face Transformers, Mistral-7B-v0.3, FAISS, LlamaIndex, BGE embed/rerank, squad_v2, Hugging Face Datasets, huggingface_hub, NumPy, llama_index, Python 3.14. `setup.sh` installs what is missing; Python packages go into a repo-local `.venv`.
- A Hugging Face token with access to `mistralai/Mistral-7B-v0.3`

## Run the Whole Bundle

This workload is also part of [`bundle-nvidia-ubuntu-2604`](https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2604), which holds all 32 NVIDIA Ubuntu 26.04 workloads as submodules:

```bash
git clone --recurse-submodules https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2604 /opt/benchmarks
cd /opt/benchmarks
./run_benchmark_suite.sh -w 432 --smoke     # just this workload
./run_benchmark_suite.sh --smoke            # all 32 workloads
./show_metrics.sh 432                       # this workload's runs from the ledger
```

The suite tools (`run_benchmark_suite.sh`, `get_benchmark_results.sh`, `show_ledger_runs.sh`, `show_metrics.sh`) are documented in [`gpu-bench-suite`](https://github.com/garys-gpu-benchmarks/gpu-bench-suite).

Other platforms: [AMD 24.04](https://github.com/garys-gpu-benchmarks/bundle-amd-ubuntu-2404) (101–132), [NVIDIA 24.04](https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2404) (201–232), [AMD 26.04](https://github.com/garys-gpu-benchmarks/bundle-amd-ubuntu-2604) (301–332).

## Troubleshooting

- **SSH dropped during setup:** reconnect and run `sudo bash setup.sh --assume-yes` again. It resumes; do not delete `.venv` or `.cache`.
- **Model download refused (401 or 403):** `mistralai/Mistral-7B-v0.3` is gated. Accept its license on Hugging Face, then `export HF_TOKEN=<token>` and run setup with `sudo -E bash setup.sh --assume-yes` so the token reaches it.
- **A metric reads `na`:** the run could not measure it on this host. The reason is in the run's `metrics_summary.txt` and `errors_info.txt`; see also [Notes](#notes).

## Repository Layout

```text
.
├── setup.sh                      # installs dependencies and builds the benchmark
├── run_benchmark.sh              # runs one profile, parses, validates
├── show_metrics.sh               # prints this workload's ledger rows
├── SPEC.md                       # engineering specification
├── benchmark_specification.json  # workload definition
├── config/                       # profile values and thresholds
├── scripts/                      # build, collect, parse, validate
├── docs/                         # PRD and contracts
├── tests/
├── results/                      # run outputs
└── LICENSE
```

## Continuous Integration

- [CI](.github/workflows/ci.yml) runs lint and static checks on every pull request and push to `main`. It does not run the benchmark.
- [GPU Smoke Benchmark](.github/workflows/gpu-smoke.yml) runs a profile on a self-hosted runner labeled `gpu`, `nvidia`, `ubu2604`. It only starts by hand (**Actions → GPU Smoke Benchmark → Run workflow**), so pull requests and forks cannot run it.

Both workflows call shared steps in [`shared-workflows@v1`](https://github.com/garys-gpu-benchmarks/shared-workflows).
