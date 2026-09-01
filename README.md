# K-means: Python vs C++ with pybind11

<div align="center">

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![C++](https://img.shields.io/badge/C%2B%2B-00599C?style=for-the-badge&logo=c%2B%2B&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white)
![pybind11](https://img.shields.io/badge/pybind11-2D6CC5?style=for-the-badge&logo=cpp&logoColor=white)

</div>

A compact performance-focused project that compares a pure Python k-means implementation against a compiled C++ implementation exposed through `pybind11`. The goal is to show how a numerical kernel can be moved from Python into native code while preserving a clean scientific Python workflow.

## Why this project exists

This repository demonstrates a common engineering pattern in high-performance data science:

- Python handles orchestration, experimentation, dataset generation, and benchmarking.
- C++ handles the hot inner loops where speed matters most.
- `pybind11` bridges the two layers with minimal friction.

The project is intentionally simple, reproducible, and fast to run locally so the performance tradeoff is easy to observe.

## Architecture

```text
┌───────────────────────────────┐
│ Python layer                  │
│ - benchmark orchestration     │
│ - synthetic dataset creation  │
│ - analysis and comparison     │
│ Files: kmeans.py, run_*.py   │
└───────────────┬───────────────┘
                │ NumPy arrays
                ▼
┌───────────────────────────────┐
│ Python/C++ boundary           │
│ - pybind11 extension module   │
│ - argument validation         │
│ Files: setup.py, pyproject.toml │
└───────────────┬───────────────┘
                │ compiled calls
                ▼
┌───────────────────────────────┐
│ C++ kernel                    │
│ - centroid assignment         │
│ - centroid updates            │
│ - lower-level numerical loops │
│ File: kmeans_cpp.cpp          │
└───────────────────────────────┘
```

Key design choices:

- Input data is passed as dense `float64` NumPy arrays.
- The C++ implementation is built as a Python extension module.
- Performance-sensitive loop logic is isolated from Python overhead.
- Benchmarking is deterministic for fixed seeds and datasets.

## Performance profile

The repository includes benchmark scripts that compare the Python and C++ implementations on the same dataset. A representative result for a benchmark configuration such as `k=25`, `n_features=10`, and `N=16250` is roughly:

| Implementation | Runtime |
| --- | ---: |
| Python (NumPy / naive reference) | ~19.87 s |
| C++ via pybind11 | ~0.064 s |

These values vary by machine, compiler, and CPU architecture, but the observed trend is consistent: the compiled code reduces the main computational bottleneck dramatically.

## Profiling and diagnostics

This repo includes profiling entry points to inspect where time is spent inside the workloads:

```bash
python profile_python.py
python profile_cpp.py
```

The scripts use Python's built-in profiling tools to reveal the dominant cost centers in the Python path, while the C++ path is measured separately.

## Quick start

### 1. Create a virtual environment

```bash
python -m venv .venv
source .venv/bin/activate
```

### 2. Install dependencies

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m pip install -e .
```

### 3. Run the k-means benchmark

```bash
python run_kmeans.py
```

### 4. Launch the web app

```bash
uvicorn web_app:app --reload
```

Then open the local FastAPI page in a browser to visualize the clustering benchmark.

## Project structure

```text
.
├── kmeans.py                 # Python reference implementation
├── kmeans_cpp.cpp           # C++ k-means kernel with pybind11 bindings
├── dbscan.py                # DBSCAN reference implementation
├── dbscan_cpp.cpp           # Optional C++ DBSCAN extension source
├── setup.py                 # Extension build configuration
├── pyproject.toml           # Modern build metadata
├── requirements.txt         # Python dependencies
├── run_kmeans.py            # CLI benchmark runner
├── run_dbscan.py            # DBSCAN runner (secondary reference)
├── profile_python.py        # Python-side profiler
├── profile_cpp.py           # C++-comparison profiling entry
├── web_app.py               # FastAPI web interface and benchmark API
├── static/                  # Frontend assets for the web app
│   ├── index.html
│   ├── app.js
│   └── style.css
├── README.md
└── .gitignore
```

## Reproducibility and behavior

- Dataset generation uses fixed seeds for repeatable experiments.
- Both implementations are compared under the same input conditions.
- Minor differences in local minima may appear due to floating-point behavior and centroid tie-breaking.
- The focus is on throughput and performance characteristics rather than exact identical cluster assignments.

## DBSCAN reference

This repo also contains a DBSCAN implementation alongside the k-means work. It is retained as a reference for a different clustering algorithm and as a template for extension-module structure.

## Contributors

- [hum4nBeing](https://github.com/hum4nBeing)
- [adarshsingh2951](https://github.com/adarshsingh2951)

## License

This project is provided as a learning and benchmarking repository for experimentation, performance analysis, and applied clustering workflows.
