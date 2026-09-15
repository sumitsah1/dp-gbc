# dp-gbc

Code for DP-GBC, a differentially private granular-ball synthesizer for tabular data.

## Privacy accounting (current code)

The repository implements the mechanism described by Proposition 1 of the paper:

    ε = D · (ε_count + ε_split) + ε_release

Fresh private class-count queries are issued only at internal nodes (depth < D).
Nodes at depth D become leaves and inherit the parent's already-noised class counts
(post-processing, zero additional budget).

Main experiment settings:

    D = 3
    ε_count = 0.1
    ε_split = 0.1
    ε_release = 2.4
    ε = 3 · (0.1 + 0.1) + 2.4 = 3.0

## Note on the published results

The experiments reported in the paper were produced by an **earlier version** of
the DP-GBC builder that issued a fresh Laplace class-count query at depth-D leaves
*in addition to* internal nodes. Its actual accounted budget was:

    ε = (D + 1) · ε_count + D · ε_split + ε_release
      = 4 · 0.1 + 3 · 0.1 + 2.4
      = 3.1

not the 3.0 stated in Proposition 1 of the paper. The paper's Proposition 1
formula describes the corrected mechanism, which is what this repository now
implements.

The corrected mechanism is **not bit-identical** to the one that produced the
published numbers, because depth-D leaf label distributions now come from the
parent node.

## Code path

- `dp_generate_granular_balls_with_radius` — DP-GBC builder, aligned with Proposition 1.
- `dp_generate_kdtree_balls` — KD-tree-style label-oblivious split baseline,
  using the same accounting convention.
- `run_dp_gbc_pipeline` — charges `D · (ε_count + ε_split) + ε_release` through `PrivacyLedger`.
- `run_dp_localclip_pipeline` — alias for `run_dp_gbc_pipeline`.

## Additional notes

- Public per-feature bounds are declared in `PUBLIC_BOUNDS` from official UCI/OpenML
  documentation; scaling to [0, 1] consumes no privacy budget.
- Synthetic sampling from released ball geometry is post-processing (zero extra budget).
- The DP-SGD baseline is binary-only and uses zCDP to (ε, δ)-DP accounting.
