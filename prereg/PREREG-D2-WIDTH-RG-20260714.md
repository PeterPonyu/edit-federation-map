# Pre-registration: edit-federation study — merging RG boundary vs WIDTH, family held fixed — 2026-07-14

> Written BEFORE launch (BUILD-ONLY authoring pass; no GPU runs, no network, no launches by
> this pass). Companion to `PREREG-RG-DEPTH-2026-07-12.md` (which varied LAYER within one
> model). This tier varies MODEL WIDTH while holding architecture family fixed, to
> deconfound the two pieces of existing evidence:
>   - Llama-3.2-1B (16L) @ L12 (75% depth): RG PASS, qualifying g ∈ {2,3,5} only
>     (an internal kill-gate note).
>   - Mistral-7B-v0.3 (32L) @ L24 (75% depth): RG PASS, **all** g ∈ {2,3,5,10,20} qualify
>     (an internal scale-transfer note).
> The apparent "boundary relaxes at scale" reading of those two cells CONFOUNDS width
> (1B→7B) with family (Llama→Mistral). this study reruns the same RG gate within ONE family across
> three widths (primary: Qwen2.5 1.5B→7B→14B) plus a second, 2-point family check
> (Llama-3.2-1B vs Llama-3.1-8B) to see whether the boundary-widens-with-scale reading
> survives family-fixing.

## Differentiation from 2601.22285 "Demystifying Mergeability"
That paper characterizes **performance degradation** under model merging as a function of
scale/mergeability heuristics (a capability-preservation question). this study asks a different,
narrower question: at what merged-group size does the **closed-form geometry discriminant**
(partial Spearman rho(I_cos, drop | I_mag) from `merging_m0.py`, the exact rank-one
cross-term identity — see that file's module docstring) stop adding predictive signal
beyond raw magnitude. This is a validity-of-the-screening-statistic boundary, not a
performance-scaling curve; no existing paper (per the 07-14 rescout) reports it holding
family fixed.

## Design

### Model list + LAYER RULE (fixed a priori: 75% relative depth, `floor(num_hidden_layers * 0.75)`)
Same rule already used for every existing RG cell (Llama-3.2-1B L12 = floor(16*.75)=12;
Mistral-7B L24 = floor(32*.75)=24) — this study does not introduce a new depth convention, only
applies the existing one to new widths. `num_hidden_layers` verified from each model's
`config.json` (Llama-3.2-1B, Llama-3.1-8B locally; Qwen2.5-1.5B locally; Qwen2.5-7B/14B via
the HF Hub API, since those checkpoints live only on the shared GPU box disk):

| Model | Family | Tier | nL | 75%-depth layer | Status |
|---|---|---:|---:|---:|---|
| Llama-3.2-1B | Llama | 1B | 16 | **L12** | EXISTING (`Llama-3.2-1B_L12_RG/`, RG PASS g∈{2,3,5}) — cite, DO NOT rerun |
| Qwen2.5-1.5B | Qwen2.5 | 1.5B | 28 | **L21** | NEW — local 24 GB GPU |
| Qwen2.5-7B | Qwen2.5 | 7B | 28 | **L21** | NEW — the shared GPU box |
| Qwen2.5-14B | Qwen2.5 | 14B | 48 | **L36** | NEW — the shared GPU box (same layer index the existing `qwen25_14b` gate band already used at its 75%-depth point — different experiment/editor, npz not reusable, but confirms the layer choice is consistent with the 07-13 wave's own banding) |
| Llama-3.1-8B | Llama | 8B | 32 | **L24** | NEW — the shared GPU box (the existing precision-twin cell used L16 = 50% depth for a *different* purpose — NOT reusable here) |
| Mistral-7B-v0.3 | Mistral | 7B | 32 | L24 | EXISTING (`Mistral-7B-v0.3_L24_RG/`, RG PASS g∈{2,3,5,10,20}) — reference only, this is the CONFOUNDED cell this study exists to deconfound; cite, DO NOT rerun |

Qwen2.5-1.5B and Qwen2.5-7B land on the SAME layer index (L21) because they share nL=28 —
this is a coincidence of the Qwen2.5 depth schedule (0.5B/1.5B/3B/7B all ship 28 layers;
only 14B+ deepens), not a driver bug; the 14B cell correctly moves to L36. Because two Qwen
cells share a layer number, output paths MUST be tagged by model, not layer alone (the
driver enforces this — see below).

### Group sizes and seeds
- Seeds: `0,1,2` (unchanged convention).
- Group sizes: driver default matches the original L12 gate's full sweep,
  `2,3,5,10,20,50,100`. For the **cross-model comparability claim**, only the common subset
  `g ∈ {2,3,5,10,20}` is quotable (this is what every existing cell — 1B and 7B-Mistral — has
  measured); `g=50,100` are exploratory bonus points, cheap on the 1.5B/local cell only.
  **Recommend launching the 14B cell with `RG_GROUP_SIZES=2,3,5,10,20` explicitly** to control
  the shared GPU box cost (see VRAM/cost table); 7B and 8B may do the same or the full sweep at the
  operator's discretion — it does not change the width-ordering test, which only reads the
  common subset.

### Decision rule (identical metric/pass-rule to the L12/L8/L14 gates — UNCHANGED)
Per cell, `merging_m0.py --rg` computes `PRE_REG_PASS_RULE` verbatim:
> PASS iff partial rho(I_cos, drop | I_mag) >= 0.15 in >= 2 non-negligible, c2-coherent
> (rho(I_cos,drop) >= 0.30) group sizes across >= 2 seeds; own-magnitude-partialling
> survival required at qualifying cells.
this study adds NO new per-cell threshold. The `per_g_summary[g].qualifies` boolean (already emitted
by `analyze_rg`) is the unit the width-ordering hypothesis below is built from — no new
metric code needed.

### Width-boundary hypothesis (preregistered ORDERING prediction, stated before any this study run)
Let `boundary(model) = max({g : per_g_summary[str(g)]["qualifies"] == true})` (the
geometry-VALID ceiling — see binding wording below; NOT the saturation boundary).

**Review fix (2026-07-14, APPROVE-WITH-FIXES): `boundary(model)` for a testable-but-all-fail
(KILL) cell.** The `max` above is undefined when the qualifying set is empty — which happens
whenever `analyze_rg`'s own verdict for that cell is `KILL` (testable cells exist, i.e. some
non-negligible/non-saturated/c2-coherent g was measurable, but none of them clears
`partial_min`). Define, for exactly that case: **`boundary(model) := 0`** ("none qualifies" —
0 is not a valid group size, so it orders strictly below every real `boundary` value g>=2 and
keeps the H-Qwen/H-Llama `<=` comparisons well-defined without a special case in the
inequality itself). This is DIFFERENT from `INCONCLUSIVE` (below), where boundary is
UNDEFINED/not reported at all because no cell was even testable.

- **H-Qwen (primary, 3-point)**: `boundary(Qwen2.5-1.5B) <= boundary(Qwen2.5-7B) <=
  boundary(Qwen2.5-14B)` — the geometry-valid federation-size ceiling is non-decreasing with
  width WITHIN one family, replicating the 1B-Llama→7B-Mistral widening (g=5 → g=20) but
  with the family confound removed.
- **H-Llama (secondary, 2-point)**: `boundary(Llama-3.2-1B) <= boundary(Llama-3.1-8B)` — same
  logic, weaker evidence (2 points, no interior control), but tests whether the widening
  replicates in the SAME family whose 1B point is already the confounded anchor.

**Both outcomes are informative (no kill-power over the existing per-cell PASS artifacts)**:
- Ordering HOLDS at both families → strengthens the scale claim into a controlled result
  (family-independent, publishable as the headline of the this study section).
- Ordering FAILS (flat or non-monotone, e.g. `boundary(7B) < boundary(1.5B)`, INCLUDING the
  KILL case `boundary(model)=0` above a smaller model's nonzero boundary) → the
  boundary-widens-with-scale reading is FAMILY-DRIVEN, not WIDTH-driven, and must be dropped
  from the federation paper's claims — but this does not touch the per-cell PASS/KILL/MIXED
  verdicts, which remain independently valid (each cell is still its own preregistered gate
  via `PRE_REG_PASS_RULE`).
- Any cell returns `INCONCLUSIVE` (per `analyze_rg`'s own verdict, i.e. no testable
  non-saturated non-negligible c2-coherent cell exists AT ALL — a strictly narrower condition
  than KILL, which requires testable cells to exist and then fail) → `boundary(model)` is
  UNDEFINED for that point, NOT 0; that model point is reported standalone with no boundary
  value, and the width-ordering claim is scoped to the remaining testable points (report
  which comparisons are still live).

### Binding wording (carried over verbatim from the L8/L14 depth prereg — applies per model)
Two boundaries per cell, never conflated:
- **geometry-VALID boundary** = `boundary(model)` above (largest `qualifies=True` g) — what
  H-Qwen/H-Llama are stated in terms of.
- **damage-still-gradated boundary** = `scoped_federation_boundary.conservative_min_across_seeds`
  (largest non-saturated g, i.e. where the merge outcome hasn't maxed out argmax-loss) —
  a DIFFERENT, looser boundary already emitted by `analyze_rg`. Report both per cell; the
  width hypothesis is about the first only.

## Compute feasibility (fp32 VRAM table)

`merging_m0.py` is fp32-only by construction (`_load_edit_model` always loads
`dtype=torch.float32`, matching the ROME value-opt fp32 rule — verified by reading
`experiments/merging_m0.py:614` directly, not assumed). Parameter counts verified via the HF
Hub API (`Qwen/Qwen2.5-{1.5B,7B,14B}`) and local `config.json`/safetensors index
(`Llama-3.1-8B`); fp32 weight footprint = `params * 4 bytes`. Activation memory is NOT the
bottleneck: ROME's value-optimization backward pass only spans from the edit layer to the
output — at 75% depth that is just the LAST 25% of the network, i.e. *less* backward span
than several already-completed wave cells (e.g. the L16=50%-depth `llama31_8b` precision-twin
cell on the same the shared GPU box), and the RG group-merge measurement phase is forward-only. So the
weight footprint below is the dominant, and already conservative, VRAM driver.

| Model | Params | fp32 weights | Target card | Fit | Precedent |
|---|---:|---:|---|---|---|
| Qwen2.5-1.5B | 1.544B | 6.2 GB | a local 24 GB GPU | Yes, ~18 GB headroom | Llama-3.2-1B fp32 (~5.0 GB) already runs full RG sweeps on this exact local card |
| Qwen2.5-7B | 7.616B | 30.5 GB | the shared GPU box (96 GB), solo | Yes | Mistral-7B-v0.3 fp32 (~29 GB) already ran a full RG sweep on this card — even CONCURRENTLY with a second fp32 8B lane (a concurrent 8B worker), i.e. ~61 GB resident simultaneously and it fit |
| Qwen2.5-14B | 14.770B | 59.1 GB | the shared GPU box (96 GB), **solo only** | Yes if run alone; do NOT co-schedule with the 7B or 8B cell (59+30=89 GB leaves too little headroom for activations/CUDA context alongside another fp32 model) | new tier for this box; 14B ran fine in bf16 (28 GB) during the 07-13 wave, so fp32 (59 GB, ~2x) is the natural extrapolation and still well under 96 GB |
| Llama-3.1-8B | 8.030B (16,060,522,496 B @ bf16 on disk / 2) | 32.1 GB | the shared GPU box (96 GB), solo | Yes | fp32 8B already loaded successfully on this card (`a concurrent 8B worker`, L16) |

Recommended scheduling on the shared GPU box (avoids the tight-headroom 14B+other case above):
run Qwen2.5-7B and Llama-3.1-8B concurrently if desired (30.5+32.1=62.6 GB, same order as
the already-proven 61 GB 8B+merging concurrent pair), then Qwen2.5-14B SOLO in its own
phase.

**Ops note (review fix, 2026-07-14): the 14B-solo constraint is operator discipline, not
software-enforced.** `run_merging_width.sh` has no VRAM introspection and does not block a
concurrent launch that would co-schedule the 14B cell with the 7B/8B cell — nothing in the
driver checks `nvidia-smi` memory headroom before starting, only whether the GPU is IDLE at
launch time (the standard util<25&&mem<1500 gate, which passes trivially if invoked before
any other lane has started). The "run 14B solo" rule above must be honored by whoever
sequences the shared GPU box phases (i.e. in the operator runbook's launch order), not assumed enforced by
the script. This will be mirrored explicitly in the operator boot runbook as an ordering
requirement, not a driver feature.

## GPU-time estimate (informational only — no launch by this pass)
The measured Llama-3.2-1B (1.24B params) full 3-seed × 7-group-size RG sweep cost ~5-8
GPU-min (see `run_merging_rg.sh` header, "HONEST GPU ESTIMATE"). Cost scales roughly with
model FLOPs (~params, since sequence length/steps are fixed): Qwen2.5-1.5B ~1.2x that cost
(~10 min); Qwen2.5-7B/Llama-3.1-8B ~6-6.5x (~35-50 min each, restricted to the 5-group-size
comparability subset); Qwen2.5-14B ~12x (~65-95 min). All comfortably inside a single
the shared GPU box restart window alongside other planned tiers (see the internal compute plan, §C, a modest budget class).

## Launch commands (reference only — NOT executed by this authoring pass)
```bash
# local, Qwen2.5-1.5B — cheap, full group-size sweep
cd edit-harness
MODEL_DIR=data/models/Qwen2.5-1.5B MODEL_TAG=qwen15b \
  RG_SEEDS=0,1,2 RG_GROUP_SIZES=2,3,5,10,20,50,100 BUDGET_MIN=60 \
  ./run_merging_width.sh

# the shared GPU box, Qwen2.5-7B (comparability subset)
PY=<remote-box>/python MODEL_DIR=<remote-box>/models/Qwen2.5-7B MODEL_TAG=qwen7b \
  RG_SEEDS=0,1,2 RG_GROUP_SIZES=2,3,5,10,20 BUDGET_MIN=150 \
  ./run_merging_width.sh

# the shared GPU box, Qwen2.5-14B — run SOLO (see VRAM table), comparability subset
PY=<remote-box>/python MODEL_DIR=<remote-box>/models/Qwen2.5-14B MODEL_TAG=qwen14b \
  RG_SEEDS=0,1,2 RG_GROUP_SIZES=2,3,5,10,20 BUDGET_MIN=240 \
  ./run_merging_width.sh

# the shared GPU box, Llama-3.1-8B second family (L24 — NOT the existing L16 twin cell)
PY=<remote-box>/python MODEL_DIR=<remote-box>/models/Llama-3.1-8B MODEL_TAG=llama8b \
  RG_SEEDS=0,1,2 RG_GROUP_SIZES=2,3,5,10,20 BUDGET_MIN=150 \
  ./run_merging_width.sh
```
Each invocation writes to a MODEL_TAG+LAYER-qualified table path
(`results/merging/RG_operating_curve_table_<tag>_L<layer>.json`) and lets `merging_m0.py`
derive its own RG bundle dir from the real model basename
(`results/merging/<basename>_L<layer>_RG/`) — neither collides with the canonical
`Llama-3.2-1B_L12` artifact or with each other, including the Qwen2.5-1.5B/7B L21 coincidence
noted above.

**Hazard noted, FIXED 2026-07-14**: a separate cloud-wave driver (not included in this
deposit) had a `merging_worker()` that invoked `merging_m0.py --rg` WITHOUT `--table_out`,
so it defaulted to writing the top-level `results/merging/RG_operating_curve_table.json` — the
same path the canonical Llama-3.2-1B L12 gate owns. It did not collide in practice only
because the box's `--out_dir` resolves under a separate on-box path that was never synced by
that name (the results-sync script pulls the model-tagged subdir, not the top-level file).
**WARN: this hazard is now fixed** — `merging_worker()` passes an explicit
`--table_out "$RES/merging/RG_operating_curve_table_mistral7b_L24.json"` (tag+layer-namespaced,
same convention as `run_merging_width.sh`), minimal diff, no other worker touched, verified
`bash -n` clean and that a family-analysis script's `merging_reldir()` regex extraction (which
parses the unchanged `out_dir=` line) still returns the same directory. This closed box's
already-completed artifacts are untouched by the fix; it only protects a future shared-GPU-box
restart of this wave from ever clobbering the canonical L12 table. Note: a cleanup script (not included) had the identical unguarded `merging_m0.py --rg` invocation (same missing
`--table_out`) — ALSO FIXED 2026-07-14 (same tag+layer convention,
`RG_operating_curve_table_mistral7b_L${layer}.json`, minimal diff, `bash -n` clean). Both
known unguarded `--rg` call sites are now namespaced.

## Analysis (CPU-only, after all cells land — not part of this authoring pass)
Standalone re-analysis of any bundle is always available with no model/GPU:
`merging_m0.py --rg_phase2_dir results/merging/<basename>_L<layer>_RG`. A future CPU pass
tabulates `boundary(model)` per cell into one table and evaluates H-Qwen/H-Llama directly —
no new script is required beyond what `analyze_rg` already emits per cell; the width-ordering
check itself is a 3-line comparison over `per_g_summary[g].qualifies`, deliberately left as
a follow-up CPU step rather than pre-built, so it is computed on REAL landed data, not
speculative code.
