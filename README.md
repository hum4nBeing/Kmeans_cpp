# K-means: Python vs C++ (pybind11)

## Purpose

This repository presents a hybrid Python/C++ architecture for performance-critical scientific workloads:

- **Python** is used for orchestration, data generation, and experiment iteration.
- **C++** is used for the computational kernel (the computationally heavy k-means inner loops).
- **`pybind11`** provides a thin binding layer so the C++ kernel can be seamlessly called from Python.

The intent is to make the boundary between experimentation code and performance-sensitive kernels explicit, demonstrating massive speedups when translating hot loops to compiled code.

## Architecture Overview

```text
┌──────────────────────────────┐
│ Python layer                 │
│  - dataset generation        │
│  - experiment orchestration  │
│  Files: kmeans.py            │
└───────────────┬──────────────┘
                │ (NumPy arrays)
┌───────────────▼──────────────┐
│ Binding layer (pybind11)     │
│  - marshaling + validation   │
│  Files: setup.py, kmeans_cpp │
└───────────────┬──────────────┘
                │ (C-contiguous float64)
┌───────────────▼──────────────┐
│ C++ layer (kernel)           │
│  - assignment + update loops │
│  - minimal allocations       │
│  File: kmeans_cpp.cpp        │
└──────────────────────────────┘
```

Key boundary decisions:
- The Python API passes a single dense `float64` array `X` into C++.
- The C++ implementation releases the Python GIL during the hot loop to allow true parallelism where possible.
- Both implementations expose comparable outputs (labels, centers, inertia, iteration count).

## Performance Snapshot

By shifting the heavy lifting to C++, the runtime drops drastically. Here is an example wall-time result from a single run for a dataset with `k=25`, `n_features=10`, and `N = 16,250`:

| Implementation | Dataset | Runtime |
| --- | --- | ---: |
| Python (Naive) | `N=16250, d=10, k=25` | ~19.87 s |
| C++ (pybind11) | `N=16250, d=10, k=25` | ~0.064 s |

*(Note: Numbers vary by machine and compiler flags)*

## Profiling Evidence

This repository includes lightweight profiling scripts to demonstrate *why* C++ is needed for the inner loops. The Python profiling often reveals that array assignment steps and nested `numpy.dot` calls dominate the runtime.

Run the profilers locally:

```bash
python profile_python.py
python profile_cpp.py
```

## Quickstart

### 1. Install & Build the Extension

First, ensure you have the requirements, and then build the C++ extension in editable mode:

```bash
python -m pip install -r requirements.txt
python -m pip install -e .
```

### 2. Run the Benchmark Scripts

To run the standalone CLI scripts and benchmark the performance:

```bash
python run_kmeans.py
```

## Repository Structure

- `kmeans.py`: Reference Python implementation of k-means (baseline + profiling target).
- `kmeans_cpp.cpp`: C++ k-means kernel (assignment + update loops) exposed via `pybind11`.
- `setup.py`, `pyproject.toml`: Packaging/build configuration for the extension.
- `profile_python.py`, `profile_cpp.py`: Reproducible profiling/timing entrypoints.
- `run_kmeans.py`: CLI runner for local timing experiments.

## Reproducibility

- The benchmark dataset generation uses a fixed `random_state`.
- Each implementation is deterministic given a fixed seed and fixed inputs.
- Due to floating-point and tie-breaking differences, Python and C++ may converge to slightly different local minima; this project focuses on timing and comparable convergence behavior.

## Legacy / Reference Components
 
This repository also contains DBSCAN code (`dbscan.py`, `run_dbscan.py`, and optional `dbscan_cpp.cpp`) kept as a secondary reference for clustering algorithms and extension-module structure. It is not used by the k-means implementation.
