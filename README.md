# Code and data for: *When Do Rank-One Knowledge Edits Merge? A Gain-Screened Two-Regime Law of Edit Federation*

This archive contains the experiment code, edit vectors, and result artifacts that
support every number, table, and figure in the manuscript. It is the research-data
deposit referenced by the paper's *Data availability* statement.

The study asks: when several independently computed rank-one knowledge edits (ROME and
its variants) are merged by plain task-arithmetic addition, at what merged-group size
does a closed-form **key-geometry** discriminant stop predicting the collateral damage
one edit inflicts on another — and how does that boundary depend on model width, depth,
editor, and a scalar **perturbation-gain** screen? The headline result is a
gain-screened two-regime law evaluated over 22 ROME cells (7 model families) plus 9
editor/dataset-varied cells, ~92,800 merge observations in total.

A companion mechanism-level study (under review) established the single-edit key-geometry
account this work builds on; it is referred to only in that anonymized form here.

## Archive layout

```
zenodo-deposit/
├── README.md            — this file
├── LICENSE              — MIT (code); result artifacts released CC BY 4.0 (see below)
├── CITATION.cff         — how to cite this deposit
├── code/                — experiment + analysis source (self-contained harness subset)
│   ├── experiments/     — the 10 scripts that produce every number
│   ├── editors/         — native editor implementations (import closure)
│   ├── metrics.py       — logit/efficacy metric helpers
│   ├── tp_edit_util.py  — device/placement helpers
│   └── run_merging_*.sh — GPU drivers (rg / width / editors / kill-gate)
├── results/
│   ├── merging/         — ROME operating-curve cells + aggregates + edit vectors
│   └── merging_editors/ — editor/dataset-generality cells + edit vectors
├── prereg/              — the frozen pre-registration documents + the ledger index
└── figures/            — the R -> tikzDevice figure pipeline (reads results/)
```

## Reproducing the paper

### Environment
The results were produced with:

| package | version |
|---|---|
| Python | 3.13.7 |
| numpy | 2.2.6 |
| scipy | 1.16.2 |
| torch | 2.12.1 (CUDA 13.0 build) |
| transformers | 5.12.1 |

Figures additionally require R with `jsonlite`, `ggplot2`, and `tikzDevice`, plus a
LaTeX toolchain (`pdflatex`) for the standalone PDFs.

### Two-phase measurement (`experiments/merging_m0.py`)
Every ROME cell is produced by one script in two phases:
- **Phase 1** (`--killgate`) installs the edits, dumps the per-edit key/value/residual
  vectors (`<cell>_s{seed}/phase1_vectors.npz`) and the closed-form kill-gate table
  (`M0_killgate_table.json`).
- **RG phase** (`--rg`) merges edits at group sizes g ∈ {2,3,5,10,20,50,100}, measures
  the per-observation drops, and writes the operating curve
  (`<cell>_L<layer>_RG/` with `rg_measurements.npz`, `rg_seed{0,1,2}_vectors.npz`,
  `rg_meta.json`, `RG_operating_curve_table*.json`).

`experiments/merging_editors.py` is the editor-general analogue (MEMIT, AlphaEdit; ROME
reproduces `merging_m0.py` to fp64 as an equivalence anchor). The `run_merging_*.sh`
drivers wrap these with the GPU-idle gate and self-test smokes. **Model weights and the
CounterFact / zsRE edit files are NOT redistributed here** (they are third-party
resources); the drivers expect a model directory via `MODEL_DIR=` and `data/counterfact.json`
(and `data/zsre.json` for the zsRE arm) present in the harness root. All analysis scripts
below run CPU-only against the shipped `results/` artifacts with no model or GPU needed.

### Paper element → script → artifact map
All artifact paths are under `results/merging/` unless noted.

| Paper element | Analysis script | Source artifact(s) |
|---|---|---|
| Operating-map table (all 22 cells, frozen gate) | `experiments/rg_map_evidence_consolidate.py` over per-cell `merging_m0.py --rg` output | `RG_operating_curve_table_*.json`, per-cell `*_L*_RG/`, `RG_map_evidence_20260716.json` |
| Gate-evidence figure (figE) and dose-response figure (figD) | `figures/make_figures.R` | `RG_map_evidence_20260716.json` |
| Gain-screen figure (figA) | `figures/make_figures.R`; gain table by `experiments/rg_gain_law.py` | `RG_gain_law_20260715.json`, `RG_gain_holdout_20260716.json` |
| g-resolved cross-talk figure (figB) | `figures/make_figures.R`; alignment by `experiments/rg_crossterm_alignment.py` | `RG_crossterm_alignment_20260715.json`, `RG_gain_law_20260715.json` |
| Gain-screen table | `experiments/rg_gain_law.py` | `RG_gain_law_20260715.json` |
| Ordering-claim Spearman (22 cells) + g=2 fractions | `experiments/rg_gain_law.py`, `experiments/rg_gain_holdout.py` | `RG_gain_law_20260715.json`, `RG_gain_holdout_20260716.json`, `RG_g2_rho_all22_20260716.json` |
| Ordering-claim permutation null | `experiments/rg_permutation_null.py` | `RG_permutation_null_20260716.json`, `perm_null_allcells/` |
| Editor/dataset-generality table | `experiments/merging_editors.py --rg`, `experiments/rg_gain_law_editors.py` | `../merging_editors/*_RG/RG_editors_table.json`, `../merging_editors/RG_gain_law_editors_20260716.json` |
| Admission-benefit table + figure (figC) | `experiments/rg_admission_benefit.py`; figC by `make_figures.R` | `RG_admission_benefit_20260715.json` |
| Signed re-analysis (constructive-merging regime) | `experiments/rg_signed_reanalysis.py` | `RG_signed_reanalysis_20260715.json` |
| Kill-gate origin (M0) | `experiments/merging_m0.py --killgate` | `M0_killgate_table.json`, per-cell `*_s{0,1,2}/` |
| cross-term ↔ value-direction alignment | `experiments/rg_crossterm_alignment.py` | `RG_crossterm_alignment_20260715.json` |

### Rebuilding the figures
```
cd figures
Rscript make_figures.R      # reads ../results/merging/*.json, writes fig{A..E}_*.tex
make                        # standalone PDFs (needs pdflatex)
```
`results/` is a sibling of `figures/`; `make_figures.R` resolves it relative to its own
directory.

### Re-running an analysis on the shipped data
Analysis scripts are CPU-only and read the bundle directories directly, e.g.:
```
cd code
python3 experiments/rg_gain_law.py            # rebuilds the gain-law table from *_RG/ bundles
python3 experiments/merging_m0.py --rg_phase2_dir ../results/merging/Llama-3.2-1B_L12_RG
```
(Run analysis scripts from `code/` so the `experiments/`, `editors/`, `metrics.py`
import layout resolves. Some scripts default to writing under a local `results/merging/`;
point them at this archive's `results/` or copy the bundles in as needed.)

## Pre-registration
`prereg/` contains the four frozen pre-registration documents named in the paper's
pre-registration ledger, plus `LEDGER-PREREG-2026-07-16.md`, which indexes each frozen
prediction to its outcome. Per the ledger's anonymity note, internal project codenames
and references to the companion manuscript under review have been masked in these
documents; **every date, threshold, directional prediction, and numeric band is preserved
verbatim.**

## Licensing
- **Code** (`code/`, `figures/`): MIT — see `LICENSE`.
- **Result artifacts** (`results/`) and **pre-registration documents** (`prereg/`):
  released under Creative Commons Attribution 4.0 (CC BY 4.0).

Model checkpoints and the CounterFact / zsRE datasets are third-party resources and are
not included or relicensed here.

## Full archive (Zenodo)

The complete archive, including the per-cell edit-vector and measurement `.npz` payloads (~2.6 GB), is deposited at Zenodo: https://doi.org/10.5281/zenodo.21405273. This repository carries the code, pre-registration documents, figure pipeline, and all aggregate result JSONs; every table and figure in the paper reproduces from these.
