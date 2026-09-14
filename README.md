# dp-gbc
Code for DP-GBC, a differentially private granular-ball synthesizer for tabular data.

## Privacy accounting

The canonical implementation follows Proposition 1:

    ε = D · (ε_count + ε_split) + ε_release

Fresh private class-count queries are made only at internal nodes (depth < D).
Nodes at depth D become leaves and inherit the parent's already-noised class counts,
so no additional raw-data count query is issued at depth D.

## Main experiment settings

    D = 3
    ε_count = 0.1
    ε_split = 0.1
    ε_release = 2.4

    ε = 3 · (0.1 + 0.1) + 2.4 = 3.0

## Alternative accounting

If a fresh private class-count query were issued at every depth-D leaf
(in addition to the internal nodes), the total would be:

    ε = (D + 1) · ε_count + D · ε_split + ε_release
      = 4 · 0.1 + 3 · 0.1 + 2.4
      = 3.1

The implementation in this repository follows Proposition 1 (the 3.0 case).

## Code path

- `dp_generate_granular_balls_with_radius` — DP-GBC builder, aligned with Proposition 1.
- `dp_generate_kdtree_balls` — KD-tree-style label-oblivious split baseline,
  using the same accounting convention.
- `run_dp_gbc_pipeline` — charges `D · (ε_count + ε_split) + ε_release` through `PrivacyLedger`.
- `run_dp_localclip_pipeline` — alias for `run_dp_gbc_pipeline`; produces the same
  DP synthetic release and is kept for backward compatibility with earlier call sites.

## Additional notes

- Public per-feature bounds are declared in `PUBLIC_BOUNDS` from official UCI/OpenML
  documentation; scaling to [0, 1] consumes no privacy budget.
- Synthetic sampling from released ball geometry is post-processing (zero extra budget).
- The DP-SGD baseline is binary-only and uses zCDP to (ε, δ)-DP accounting.
