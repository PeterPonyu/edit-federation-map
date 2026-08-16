# Edit federation map

Public measurements of key geometry and collateral interference when rank-one locate-then-edit updates are merged by task arithmetic.

**[Map](https://peterponyu.github.io/edit-federation-map/)** · **[Source](https://github.com/PeterPonyu/edit-federation-map)** · **[Numerical archive](https://doi.org/10.5281/zenodo.21405273)**

## The object

Locate-then-edit methods write a rank-one update into an MLP. Merging several such updates by plain addition produces collateral interference: one fact moves another. A closed-form key-cosine discriminant tracks that damage until perturbation gain and merge size push the system into a second regime.

Gain is a rank-level screen, never a carrier. Geometry tracks the size of the cross-term; it does not anti-predict interference.

## Frozen measurement object

- **22 ROME cells** across 7 families; 65,868 merge observations.
- **Llama-2-13B is an addendum and is excluded from the n=22 freeze.**
- Ordering Spearman(gain, constructive fraction) prints **−0.82**. The frozen object is **≤ −0.7**. The point estimate is not the frozen bound.
- Gain cut at 8 (13 high-gain / 9 low-gain).
- Qwen2.5-14B constructive fraction **81–88%**.
- The geometry-valid **g ≤ 5** window is the **reference cell only**; other cells can qualify at larger g. Do not read g ≤ 5 as a map-wide rule.
- High-gain admission benefit **+0.647** is a member-target, fixed-composition, retrospective quantity, not a prospective admission rule.
- Prospective admission stays **Mixed / not admitted** and is not a deployable admission rule.

Live figures and the 22-cell table: https://peterponyu.github.io/edit-federation-map/measurements/

## Reproduce

The GitHub tree and the Zenodo archive carry experiment code, edit vectors, per-cell operating curves, and the R figure pipeline. Model checkpoints and the CounterFact / zsRE fact files are third-party and are not redistributed.

From the archive root, make the result arrays visible to the analysis tree, then rebuild the operating map, signed reanalysis, cross-term alignment, matched-dose span, gain screen, gain holdout, and admission-benefit tables. Public plots rebuild from the R figure pipeline. All of those rebuilds are CPU-only.

Frozen protocol documents ship in the archive under CC BY 4.0.

## Archive

Concept DOI (unchanged; v1.0.0, 2026-07-17): https://doi.org/10.5281/zenodo.21405273

- Code: MIT
- Result arrays and frozen protocol documents: CC BY 4.0
