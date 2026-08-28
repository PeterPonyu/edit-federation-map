# Edit federation map

When do rank-one knowledge edits merge? A gain-screened two-regime law of edit federation,
with the public measurements behind it.

**[Map](https://peterponyu.github.io/edit-federation-map/)** ·
**[Source](https://github.com/PeterPonyu/edit-federation-map)** ·
**[Numerical archive](https://doi.org/10.5281/zenodo.21405273)**

## The result

Independently computed weight updates are commonly assumed to interfere monotonically.
Rank-one knowledge edits do not. Aligned merge cross-talk reverses sign: high-gain layers
damage member-target logits, while low-gain layers raise them, before a crossover whose
location varies by architecture.

Perturbation gain ranks the two regimes, so it works as a rank-level screen: it tells you
which regime a layer is in, and inside a regime a closed-form key-cosine discriminant tracks
the size of the cross-term. Both quantities are read from the editor's own closed form, so
neither requires a learned probe or a held-out fit.

## What is measured

A frozen measurement object, released whole:

- **22 ROME cells** across 7 architecture families spanning 1B to 20B, and **65,868 merge
  observations**.
- Ordering **Spearman(gain, constructive fraction) = −0.82**, against a frozen directional
  bound of ≤ −0.7; family-clustered 95% CI **[−0.88, −0.51]**.
- Gain cut at **8**, splitting the map into **13 high-gain** and **9 low-gain** cells.
- Constructive merges dominate the low-gain regime: Qwen2.5-14B reaches a constructive
  fraction of **81–88%**.
- Constructive cross-talk survives matching on solo installation and residual magnitude at
  Mistral-Nemo-12B, and reappears at a **held-out Llama-2-13B cell** outside the n=22 freeze.
- Per-cell geometry-valid windows ship in the archive: **g ≤ 5** at the reference cell, and
  larger at six further qualifying cells.
- Geometry-ordered selection at a 25% budget avoids **+0.647** member-target logits of drop
  versus random under fixed composition.
- A **prospective group-formation test** at Llama-3.2-1B L12 supports the primary geometry
  comparisons in all three seeds.

Live figures and the 22-cell table: <https://peterponyu.github.io/edit-federation-map/measurements/>

## Reproduce

The GitHub tree and the Zenodo archive carry the experiment code, the edit vectors, the
per-cell operating curves, and the R figure pipeline.

From the archive root, make the result arrays visible to the analysis tree, then rebuild the
operating map, the signed reanalysis, cross-term alignment, matched-dose span, the gain
screen, the gain holdout, and the admission-benefit tables. Public plots rebuild from the R
figure pipeline. Every one of those rebuilds is CPU-only — no model weights required.

Model checkpoints and the CounterFact / zsRE fact files are third-party resources and are
served from their original sources rather than redistributed here.

## Scope of the claims

Stated once, so the measurements above can be read at face value. The two-regime law is
established on the frozen 22-cell object; the geometry-valid group size is a per-cell window
and the reference cell's `g ≤ 5` is not a map-wide constant. Gain is a rank-level screen
rather than a carrier of the effect. The admission benefit is a member-target quantity under
fixed composition; the prospective run supports the primary geometry comparisons, and its
utility-matched gate reads Mixed.

## Archive and license

Concept DOI (unchanged; v1.0.0, 2026-07-17): <https://doi.org/10.5281/zenodo.21405273>

- Code: MIT
- Result arrays and frozen protocol documents: CC BY 4.0
