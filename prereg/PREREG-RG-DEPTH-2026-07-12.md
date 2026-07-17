# Pre-registration: RG depth-generality tier (L8, L14) — 2026-07-12

> Written BEFORE launch. Follows the L12 RG PASS (NUMBERS-CONFIRMED 2026-07-12; see
> an internal kill-gate findings note). This tier asks one
> question: does the small-merge geometry law replicate across depth?

## Design (identical to the confirmed L12 RG; layer is the only change)
- `RG_LAYER={8,14} ./run_merging_rg.sh` — merging_m0.py --rg, Llama-3.2-1B, CounterFact,
  n_edits=200, seeds {0,1,2}, g ∈ {2,3,5,10,20,50,100}, fresh solo banks per layer/seed.
- Outputs layer-tagged (`RG_operating_curve_table_L{8,14}.json`,
  `Llama-3.2-1B_L{8,14}_RG/`); the confirmed L12 artifacts are never touched.

## Decision rule (unchanged from the L12 prereg, applied per layer)
PASS iff partial ρ(I_cos, drop | I_mag) ≥ 0.15 in ≥2 non-negligible, c2-coherent
(ρ(I_cos,drop) ≥ 0.30), non-saturated (argmax_loss ≤ 0.8) group sizes across ≥2 seeds;
own-magnitude-partialling survival required at qualifying cells.

## Pre-registered expectations (both outcomes informative; NO kill-power over L12)
- **L8** (the geometry-dominant regime documented in the companion mechanism study (under
  review)): expect the law to HOLD at small g — a replication strengthening the
  depth-generality claim.
- **L14** (the norm-growth-dominant regime documented in the companion study): geometry
  partial may ATTENUATE — if so, this mirrors the companion mechanism study's documented
  regime transition and SCOPES the
  merging law to geometry-dominant depths (a finding, not a failure).
- Failure at BOTH layers → the L12 result stands but the paper claims L12-only
  (single-depth caveat prominent).

## Binding wording carried over from L12
Two boundaries reported per layer: geometry-VALID (largest c2-passing g) and
damage-still-gradated (largest non-saturated g). Never conflate them.
