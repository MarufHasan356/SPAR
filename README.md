# SPAR: Spectral-Structural Purification with Adaptive Revalidation

This repository contains a clean reproducibility implementation of **SPAR**, a four-stage defense against graph backdoor attacks in transductive node classification.

SPAR combines:

1. **Dual-view detection** — spectral locality and feature-structure inconsistency scores identify suspicious training nodes.
2. **Loss-based purification with feature retention** — suspicious and high-loss nodes are down-weighted rather than having their attributes deleted.
3. **Adaptive revalidation** — confidence-consistency training with controlled edge perturbation reduces residual backdoor influence.
4. **Test-time sanitization** — a budgeted edge-risk mechanism removes likely trigger attachments before evaluation.

The repository also includes the official DShield and RIGBD comparison implementations under `third_party/`. SSP-only code is intentionally not included.

## Repository layout

```text
.
├── configs/                    # Dataset and sweep configurations
├── experiments/                # Reproducible experiment entry points
├── src/spar/                   # Core SPAR implementation
│   ├── attack.py               # Frozen subgraph backdoor attack
│   ├── data.py                 # Cora/CiteSeer/PubMed loading
│   ├── detection.py            # Stage 1: dual-view detector
│   ├── purification.py         # Stage 2: loss-based purification
│   ├── revalidation.py         # Stage 3: adaptive revalidation
│   ├── sanitizer.py            # Stage 4: test-time sanitizer
│   ├── models.py               # GCN, GAT, GraphSAGE
│   ├── metrics.py              # CA, ASR, DR, FPR, precision, F1
│   └── pipeline.py             # End-to-end SPAR orchestration
├── tests/                      # Metric contract tests
├── third_party/DShield/        # Official DShield baseline source
├── third_party/RIGBD/          # Official RIGBD baseline source
├── requirements.txt
├── environment.yml
└── pyproject.toml
```

## Installation

Python 3.10 or newer is recommended.

```bash
git clone <your-repository-url>
cd SPAR-Repository
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m pip install -e .
```

For CUDA, install the PyTorch and PyTorch Geometric wheels matching your CUDA version before installing the remaining requirements.

Planetoid datasets are downloaded automatically by PyTorch Geometric into `data/` on the first SPAR run.

## Main SPAR result

```bash
python experiments/run_spar.py \
  --dataset Cora \
  --architecture gcn \
  --seed 0 \
  --defense spar
```

Undefended control:

```bash
python experiments/run_spar.py \
  --dataset Cora \
  --architecture gcn \
  --seed 0 \
  --defense none
```

Supported datasets are `Cora`, `CiteSeer`, and `PubMed`. Supported architectures are `gcn`, `gat`, and `graphsage`.

## Architecture transfer sweep

The following command runs the frozen-attack transfer study:

- 3 datasets: Cora, CiteSeer, PubMed
- 3 architectures: GCN, GAT, GraphSAGE
- 3 seeds: 0, 1, 2
- 27 total runs
- GCN retained as the architecture-transfer control

```bash
python experiments/spar_architecture_sweep.py
```

The aggregate file is saved to:

```text
results/architecture_sweep.csv
```

## Ablation and component analysis

```bash
python experiments/run_ablation.py --dataset Cora --architecture gcn --seed 0
```

This evaluates:

- full SPAR
- without dual-view detection
- without loss-based purification
- without adaptive revalidation
- without test-time sanitization

## Robustness studies

Poisoning-rate sweep:

```bash
python experiments/run_robustness.py \
  --dataset Cora \
  --mode poisoning-rate
```

Trigger-size sweep:

```bash
python experiments/run_robustness.py \
  --dataset Cora \
  --mode trigger-size
```

## Efficiency measurement

```bash
python experiments/run_efficiency.py \
  --dataset Cora \
  --architecture gcn \
  --repeats 3
```

## Multi-seed validation

```bash
python experiments/run_multi_seed.py \
  --dataset Cora \
  --architecture gcn \
  --seeds 0 1 2 3 4
```

## Official baseline commands

Print the DShield command:

```bash
python experiments/run_baselines.py --method dshield --dataset Cora
```

Run DShield:

```bash
python experiments/run_baselines.py --method dshield --dataset Cora --execute
```

Run the packaged RIGBD Cora example:

```bash
python experiments/run_baselines.py --method rigbd --dataset Cora --execute
```

For RIGBD datasets beyond Cora, download the official authors' assets and update the file paths in `experiments/run_baselines.py` before execution.

## Third-party baseline assets

The full archive includes the packaged DShield and RIGBD assets. If you use the smaller web-upload archive, restore the official third-party source and large assets with:

```bash
python experiments/setup_third_party.py
```

## Metrics

Each run writes a JSON file and appends to `results/experiment_results.csv`.

- `clean_accuracy` (↑): accuracy on non-poisoned test nodes
- `attack_success_rate` (↓): fraction of poisoned test nodes predicted as the target label
- `detection_rate` (↑): recall over poisoned training nodes
- `detection_precision` (↑): precision of suspicious-node detection
- `false_positive_rate` (↓): fraction of clean training nodes flagged as suspicious
- `removed_edges`: number of edges removed by test-time sanitization

## Testing

```bash
python -m pytest tests/
```

## Reproducibility notes

- Every SPAR run fixes Python, NumPy, PyTorch, and CUDA seeds.
- The attack is frozen across architectures in the sweep.
- Trigger nodes are excluded from all train/validation/test masks.
- ASR is computed only on original target nodes selected for attack evaluation.
- Clean accuracy is computed only on non-poisoned test nodes.
- Baseline implementations remain isolated in `third_party/` so the SPAR contribution is not conflated with prior defenses.

## Third-party code

See `THIRD_PARTY_NOTICES.md` for attribution, source links, licenses, and citation information.
