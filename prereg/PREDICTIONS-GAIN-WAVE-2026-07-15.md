# FROZEN PREDICTIONS — local gain-wave (A) + pulled bundles (B), 2026-07-15 ~12:55 EDT

**Written BEFORE any A-wave GPU cell runs and BEFORE analyzing the two pulled bundles'
npz.** Descriptive program (not a pass/kill gate): testing the two-level GAIN law from
an internal signed re-analysis note — model-level perturbation gain (logit
response per unit relative value-space dose (d_a·R_a)/‖R_a‖²) sets the merge cross-talk
regime: LOW gain → constructive (frac(drop<0) high, negative geometry partials), HIGH
gain → destructive (positive partials). Dose modulates within regime.

## Gain measurement (fixed a priori)
gain := median over positive-dose observations (g≤20 pooled, 3 seeds) of |drop| / rel_dose,
with rel_dose = (d_a·R_a)/‖R_a‖². Regime stat: frac(drop<0) over the same observations.
Known anchors (already measured): Llama-1B L12 gain HIGH (destructive, frac<0 ≈ 0.03),
Mistral-7B L24 mid (transitional, ≈ 0.3), Qwen-1.5B L21 low-mid (≈ 0.45), Qwen-14B L36
LOW (constructive, ≈ 0.85).

## B — pulled bundles (npz not yet analyzed at freeze time; tables were known)
- **Llama-3.1-8B L24**: HIGH gain, destructive (frac(drop<0) < 0.2 at small g); strongly
  positive partials already known from its table (+0.29..+0.64).
- **Qwen2.5-7B L21**: gain between Qwen-1.5B and Qwen-14B; frac(drop<0) between them
  (~0.4–0.7 at g=2); partials weakly positive at small g (known table +0.09..+0.25).

## A — new local cells (75% depth, g={2,3,5,10,20}, seeds 0,1,2, fp32, n_edits=200)
Ordering prediction (the load-bearing claim): **within the A-wave, the constructiveness
ordering matches the (inverse) gain ordering** — whatever the individual values, rank
correlation between measured gain and frac(drop<0) across ALL ≥11 cells stays strongly
negative (Spearman ≤ −0.7).
Directional guesses (weaker, stated for honesty):
- **gemma-2-2b L19**: positive-sign family at small scale → HIGH-ish gain, destructive
  (frac<0 < 0.3), positive small-g partials.
- **Llama-3.2-3B L21**: interpolates Llama-1B→8B: HIGH gain, destructive (frac<0 < 0.2),
  positive partials.
- **Qwen2.5-3B L27**: interpolates Qwen-1.5B→7B: low-mid gain, transitional
  (frac<0 ≈ 0.35–0.6 at g=2).
- **Phi-3.5-mini L24**: no prior anchor in the merging series (damage-law atlas has no
  clean Phi sign) — stated as EXPLORATORY, no directional prediction; only the ordering
  claim applies.

## Falsifier
If a cell shows LOW measured gain with clearly DESTRUCTIVE behavior (or high gain with
constructive), the gain law loses its carrier status and family-specific structure returns
as the primary explanation. The decisive 12–14B non-Qwen cell (C) remains the sharpest
test either way.

## ADDENDUM — depth-contrast extension (frozen 2026-07-15 ~13:50, BEFORE launch)
The 75%-depth wave landed (all predictions held; Phi-3.5 = low-gain majority-constructive
at 3.8B). Extension: three 50%-relative-depth cells on the SAME models to test whether
the gain collapse is a DEPTH phenomenon within each affected architecture (the existing
Qwen-1.5B L14-vs-L21 contrast pattern):
- **Phi-3.5-mini L16 (50%)**: prediction — HIGH gain (≥10) and destructive
  (frac(drop<0) < 0.3), i.e., Phi mirrors the Qwen depth pattern. If instead it stays
  low-gain/constructive at 50%, the collapse is architecture-wide, not depth-gated.
- **Qwen2.5-3B L18 (50%)**: prediction — HIGH gain, destructive (mirrors Qwen-1.5B L14:
  28.1 / 0.242).
- **gemma-2-2b L13 (50%)**: prediction — HIGH gain, destructive (gemma already high-gain
  at 75%; 50% should be ≥ its L19 gain 12.5).
Ordering claim extends: Spearman(gain, frac-constructive) stays ≤ −0.7 over all ~17 cells.

## ADDENDUM 2 — GPT-2-XL exploratory pair (frozen 2026-07-15 ~14:30, BEFORE launch)
Two cells on gpt2-xl (1.5B, 48 layers, `gpt2` arch path — a pre-modern architecture
generation): **L36 (75%)** and **L24 (50%)**. EXPLORATORY — no directional prediction
(no prior anchor for GPT-2 in the merging series); only the ordering claim applies
(Spearman(gain, frac-constructive) ≤ −0.7 over all ~19 cells). Purpose: does the
depth-gated low-gain/constructive regime exist outside modern RMSNorm/SwiGLU
architectures?

## Protocol notes
Driver: `run_merging_width.sh` (reviewed 07-14; refuse-guard + idle gate util<25&&mem<1500
×3), serial chain, per-tag pids/logs, kill by PID only. Machine shared with an
unrelated CPU job — untouched. Lid open. No card needed on the
shared GPU box for this wave.
