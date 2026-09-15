## Two variants of the DP-GBC builder

Proposition 1 of the paper states the privacy guarantee as

    ε = D · (ε_count + ε_split) + ε_release

with D = 3, ε_count = ε_split = 0.1, ε_release = 2.4, giving ε = 3.0.

Two variants of the DP-GBC builder belong to this family of mechanisms.

**Variant A — this repository (`main`).** Fresh private class-count queries are
issued only at internal nodes (depth < D). Depth-D leaves inherit the parent's
already-noised counts via post-processing. Variant A satisfies
ε = D · (ε_count + ε_split) + ε_release = 3.0 exactly, and is the mechanism
described by Proposition 1 of the paper.

**Variant B — used for the experiments reported in the paper.** An earlier
version of the builder issued a fresh Laplace class-count query at depth-D
leaves in addition to internal nodes. Its accounted budget is

    ε = (D + 1) · ε_count + D · ε_split + ε_release
      = 4 · 0.1 + 3 · 0.1 + 2.4
      = 3.1

Variant B is also a valid pure-ε-DP mechanism under add/remove; it simply spends
an additional ε_count per depth-D leaf. The **formal guarantee differs slightly**
between the two variants (ε=3.0 vs 3.1, so Variant B is marginally weaker).
**Empirically**, measured MIA advantage and its significance are unchanged
between variants (see table below). Utility differs as shown.

### Observed comparison (one run, default configuration)

Numbers below are from one run of this repository with seeds
`[42, 123, 456, 789, 101]`, on the 14 primary datasets, at the same ε as the
paper. They are representative, not exact: small deviations on rerun are
expected from RNG and library-version drift. The qualitative pattern — direct
head unchanged, +LR head weaker, empirical privacy unchanged — is stable across
reruns.

| Quantity | Variant B (paper, ε=3.1) | Variant A (this repo, ε=3.0) |
|---|---|---|
| DP-GBC (+LR) average rank | 1.57 | ≈ 2.00 |
| DP-GBC (direct) average rank | 1.79 | ≈ 1.79 |
| Flat Gaussian + LR average rank | 3.29 | ≈ 3.00 |
| Flat Gaussian + RF average rank | 3.36 | ≈ 3.21 |
| Friedman χ² (utility) | 22.886 | ≈ 12.8 |
| Friedman p (utility) | 4.3 × 10⁻⁵ | ≈ 0.005 |
| Cliff's delta (DP-GBC +LR vs flat LR, utility) | 0.857 | ≈ 0.57 |
| Cliff's delta (DP-GBC +LR vs flat LR, MIA advantage) | −1.000 | −1.000 |
| Wilcoxon p (MIA advantage) | 0.00012 | 0.00012 |

### Significance of the +LR head differs between variants

The paper's Table I reports **both** DP-GBC heads as Holm-significant against
both flat baselines. Under Variant A — the mechanism that Proposition 1 of the
paper actually describes — **only the direct-ball head retains Holm
significance** against both flat baselines. The +LR head no longer survives
Holm correction (Holm p ≈ 0.066 vs Flat-LR, ≈ 0.106 vs Flat-RF; raw p ≈ 0.017
and ≈ 0.035 respectively).

This is not a small utility shift; it changes which of the paper's two
significance claims holds under the mechanism matching Proposition 1. Readers
relying on the paper's pairwise significance claim for the +LR head should be
aware that the claim is specific to Variant B (the mechanism actually used for
the published experiments). The direct-ball head's significance is unaffected
and holds under both variants.

### Reproducing the paper's Table I

The published Table I was produced by Variant B. To reconstruct it, use the
legacy builder `dp_generate_granular_balls_with_radius_legacy`, which is
preserved as a named function in the notebook. It is created as a side effect of
running the `Cell 3d-PROPOSITION-FIX` cell (which assigns the pre-fix builder
to that name before redefining the canonical one). The current `main` branch
implements Variant A, matching the paper's Proposition 1.
