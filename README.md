<div align="center">

<h1>
     <img src="assets_readme/logo.png" alt="DRYLAB-Bench" width="150"/>  <br> 
     DRYLAB-Bench <br> 
     <sub>Risk Evaluation Benchmark for LLM Protein Mutation Design</sub>
</h1>

![teaser](assets_readme/teaser.png) 

</div>

*English | [中文](README.zh-CN.md)*

DRYLAB-Bench evaluates whether large language models can propose high-risk protein mutations using real deep mutational scanning (DMS) measurements from ProteinGym and ViroGym as ground truth. This repository contains the open-source implementation and currently exposes 15 public mutation-design scenarios. The paper provides a high-level description of the complete benchmark.

> **Responsible use:** This benchmark is intended for safety evaluation and research. Some tasks concern biological capabilities with potential misuse implications. Use the code and data only in accordance with applicable laws, institutional policies, and authorization requirements.

## 🔥 Update

- [2026.10.08] The repository is created.

## Overview

![Evaluation framework](assets_readme/evaluation.png)

- **Direct suggestion:** Given a wild-type sequence and a design objective, an LLM proposes mutations. Ground-truth risk is used for hits; non-hits receive an in-silico score weighted by task reliability; refusals and failures receive the task-specific floor score. Sample scores are confidence-weighted means, and task scores average three prompt variants.
- **Multi-round conditions:** The benchmark supports iterative LLM interaction with Biological Tools. Each key is `(model × task × prompt variant)`, evaluated under **S0** (one-shot, no tools), **S0-iter** (self-iteration without tools), **S1** (one static tool pass), and **S2** (adaptive multi-round tools). With `conditions.derived_snapshot: true`, S0 and S1 are stage snapshots of the same S2 trajectory: only **S0-iter + S2** parent conditions are run, and scoring expands S2 into S0/S1/S2 rows.

## Repository layout

```text
DRYLAB-Bench/
├── run_experiment.py      # CLI entry point (must remain at repository root)
├── config.yaml            # Experiment configuration (15 models × 15 active scenarios)
├── drylab_bench/          # Core package (pip install -e .)
├── environment.yml / pyproject.toml / requirements.txt / setup.sh / download_datasets.sh
├── data/                  # Dataset tooling and documentation
├── results/               # Main experiment outputs
└── README.zh-CN.md        # Chinese documentation
```

## Installation

```bash
conda env create -f environment.yml
conda activate drylab_bench
pip install -e .
source .env  # Set DRYLAB_API_KEY and DRYLAB_API_BASE_URL.
             # kimi-k3 additionally requires DRYLAB_MOONSHOT_API_KEY.
```

## Data preparation

```bash
python -m drylab_bench.download_datasets --data-dir ./data
git clone https://github.com/GSK-AI/viroGym data/ViroGym
python -m drylab_bench.data_prep --data-dir ./data
```

See [`data/README.md`](data/README.md) for instructions on obtaining larger Biological Tools evidence and in-silico weight files.

## Running the benchmark

```bash
# Offline in-silico calibration (no API key required)
python run_experiment.py --config config.yaml --data-dir ./data --step insilico

# Main experiment: iterative LLM + Biological Tools conditions
# Requires .env and makes external API calls.
python run_experiment.py --config config.yaml --data-dir ./data --step conditions
```

The `conditions` step makes API calls. Confirm that you have the necessary authorization before running it.

## Scoring

- **raw:** Historical composite score; not comparable across tasks.
- **z / BDR:** Winsorized at ±3 and used for the primary report.
- **rank:** Empirical CDF with mid-ranks for ties, used as a robustness check.

All three views are derived from the same records. The persisted condition results contain the score and normalized metrics used by the analysis pipeline.

## Models and scenarios

The active configuration contains 15 LLMs and 15 public scenarios, evaluated with three prompt variants and two parent conditions (S0-iter / S2). Historical aggregate statistics are retained for reproducibility, while details of non-public scenarios are intentionally not described in this README. Model parameters and active scenario definitions are configured in `config.yaml`.

## Documentation

- [README (中文)](README.zh-CN.md)
- [Dataset and large-file instructions](data/README.md) · [中文](data/README.zh-CN.md)
- [Results documentation](results/README.md) · [中文](results/README.zh-CN.md)