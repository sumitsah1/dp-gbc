# dp-gbc

Code for DP-GBC, a differentially private granular-ball synthesizer for tabular data.

## Privacy accounting

The mechanism satisfies pure ε-DP under add/remove with

ε = (D + 1) · ε_count + D · ε_split + ε_release

With D = 3, ε_count = ε_split = 0.1, ε_release = 2.4, this gives ε = 3.1.

Fresh private class-count queries are issued at every node, including depth-D leaves. The split-selection exponential mechanism operates only at internal nodes. The final per-leaf size release is composed sequentially with both.

The PrivacyLedger reports the nominal budget

D · (ε_count + ε_split) + ε_release

which equals 3.0 for the main configuration. The mechanism's actual guarantee is ε = 3.1, i.e. true ε = nominal ε + ε_count. The paper discloses this distinction in Proposition 1 and in Sections III-B and IV-A.

The epsilon sweep uses the same allocation rule with TOTAL_EPS ∈ {0.5, 1.0, 3.0, 8.0}, so the offset ε_count applies to each swept budget as well.

## Code

Main functions:

- dp_generate_granular_balls_with_radius — DP-GBC builder.
- run_dp_gbc_pipeline — drives the mechanism and the PrivacyLedger.
- run_dp_localclip_pipeline — alias for run_dp_gbc_pipeline.

run_dp_localclip_pipeline is a forwarding wrapper, not a separate mechanism. In the paper, DP-GBC (direct) and DP-GBC (+LR) are the same release evaluated with two different downstream heads.

## Notes

Public per-feature bounds are declared in PUBLIC_BOUNDS from official UCI/OpenML documentation. Scaling to [0, 1] consumes no privacy budget.

Synthetic sampling from released ball geometry is post-processing, so it costs zero extra privacy budget.

eps_center_per_level and eps_radius_per_level are accepted for API compatibility but are not consumed. Centers and radii are deterministic public post-processing of already-released split thresholds, so no budget is charged against them.

The DP-SGD baseline is binary-only. It is full-batch DP-GD with zCDP to (ε, δ)-DP accounting via Bun and Steinke (2016). No Poisson subsampling or amplification-by-subsampling is applied.

The DP-CTGAN baseline is implemented via the SmartNoise-Synth library: https://github.com/opendp/smartnoise-sdk
