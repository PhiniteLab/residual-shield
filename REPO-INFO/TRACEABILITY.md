# Traceability

This repository is the companion-code artifact for arXiv:2606.01397 (`PAPER.yml`); the
manuscript LaTeX source lives in the companion repository `PhiniteLab/sharq-twin`
(`paper_of_record: false` here). The claims traced below are this repository's own, stated in
`experiments/claims/README.md` — deliberately limited to simulation behavior, not the paper's
comparative wording. The manuscript's own claim statements are traced in `sharq-twin`'s
`REPO-INFO/TRACEABILITY.md`; where those claims rely on numeric evidence, that evidence lives
in this repository's `experiments/results/`.

## Active Claims and Requirements

| ID | Type | Statement | Status |
|---|---|---|---|
| RS-1 | architectural invariant | Residual supervision preserves the fixed autopilot: Q-learning and SHARQ-HJB adjust only `[Va_c, h_c, chi_c]`; the autopilot stays the actuator-facing controller | supported |
| RS-2 | architectural invariant | The residual action space is finite and bounded (no-op, ±2 m/s airspeed, ±10 m altitude, ±3 deg heading) and is clipped before the autopilot | supported |
| RS-3 | architectural invariant | The baseline, tabular-Q, and SHARQ-HJB controller modes share the same plant, wind, actuator, autopilot, force/moment, and dynamics path | supported |
| RS-4 | method claim | The SHARQ-HJB residual supervisor scores finite candidate residuals with a semi-discrete Bellman/HJB value surrogate, Hamiltonian-style advantage, and a CLF/CBF-style finite-action risk filter, and records the selection diagnostics | supported |
| RS-5 | definitional/scope claim | `control_energy_integral` is an actuator-command activity index (not physical energy) and `safety_time_fraction` is an empirical simulator threshold fraction (not certification evidence); SHARQ-HJB is finite-action and HJB/Bellman-inspired, not a continuous HJB PDE solver | supported (definitional, holds by construction) |
| RS-6 | empirical/comparative claim | On the stored full-duration benchmark (20 episodes/mode), SHARQ-HJB has the lowest mean RMS altitude error (64.7 m vs. 114.1 m baseline, 65.8 m Q-residual) and lowest mean RMS reference error (44.8 m vs. 338.6 m baseline, 88.8 m Q-residual); online Q-learning has the lowest mean `safety_time_fraction` (0.0031 vs. 0.0048 baseline, 0.0033 SHARQ-HJB) | partially_supported |
| RS-7 | empirical/boundary claim | The full-duration ranking above does **not** hold on the short-horizon coarse sweep or the single fight-mode smoke run: on the coarse 20x50-seed sweep (1000 episodes/mode) the baseline has the lowest mean RMS reference error (0.866 m vs. 1.174 m SHARQ-HJB, 1.183 m Q-residual); on the 60 s fight-mode smoke run SHARQ-HJB has the lowest altitude RMS (68.2 m vs. 77.8 m Q-residual, 100.9 m baseline) | supported |

## Method-to-Implementation Links

| ID | Literature or Decision Basis | Method | Implementation |
|---|---|---|---|
| RS-1 | Residual/shielded-RL command-shaping design (`README.md` "central design choice") | Nominal command `r`, residual `Δr`, `r_supervised = clip(r + Δr)` computed ahead of the autopilot | `backend/src/uavsim/server.py` (`Runtime.step`), `backend/src/uavsim/autopilot.py` |
| RS-2 | Finite-action command shaping | Discrete residual action set `A = {no-op, ±2 m/s, ±10 m, ±3 deg}` | `backend/src/uavsim/q_learning.py`, `backend/src/uavsim/sharq_hjb.py` |
| RS-3 | Shared-runtime comparability requirement for a controlled benchmark | `Runtime`/`experiment_runner` dispatch on `controller_mode` over one shared step loop | `backend/src/uavsim/server.py`, `backend/src/uavsim/experiment_runner.py`, `backend/src/uavsim/dynamics.py`, `backend/src/uavsim/forces_moments.py` |
| RS-4 | Semi-discrete Bellman/HJB value surrogate + CLF/CBF-style filtering | Normalized feature vector `z`, quadratic value, Bellman/HJB value table, Hamiltonian-style advantage, risk filter over candidates | `backend/src/uavsim/sharq_hjb.py` (`SHARQHJBConfig`, `HJBCandidate`, `SHARQHJBResidualSupervisor`) |
| RS-6 | Benchmark comparison across controller modes | `run_batch` / `compare_all_methods` over the full-duration scenario set | `backend/src/uavsim/experiment_runner.py`, `scripts/sync_results.py`, `scripts/export_tables.py` |
| RS-7 | Short-horizon/seed-sensitivity and single-run cross-checks on the same runner | `run_batch` with `--seed-count 50 --duration 8` (coarse sweep) and a single 60 s `fight_mode` run | `backend/src/uavsim/experiment_runner.py` (same entry point as RS-6, different invocation flags) |

## Validation and Evidence Links

| ID | Test or Experiment | Metrics or Criteria | Evidence | Output |
|---|---|---|---|---|
| RS-1 | `test_sharq_hjb_residual_uses_hjb_metrics_and_preserves_nominal_baseline` | residual stays within bound, nominal path preserved when no-op selected | `backend/tests/test_uavsim_core.py:342` | pass (part of 41 passed, `README.md`) |
| RS-2 | `test_q_learning_residual_gate_and_shield_prevent_unneeded_heading_actions` | gated residual actions in easy conditions | `backend/tests/test_uavsim_core.py:299` | pass |
| RS-3 | `test_experiment_runner_reports_and_persists_fixed_and_q_learning_metrics`, `test_reference_benchmark_writes_method_result_folders` | fixed/Q/HJB modes produce comparable per-method result folders from one runner | `backend/tests/test_uavsim_core.py:388,430` | pass |
| RS-4 | `test_sharq_hjb_residual_uses_hjb_metrics_and_preserves_nominal_baseline`, `test_q_learning_checkpoint_round_trip_json_and_npz` (SHARQ branch) | `hjb_value`/`hjb_advantage`/`shield_active`/`candidate_count` populated and round-trip through JSON/NPZ | `backend/tests/test_uavsim_core.py:230,342` | pass |
| RS-5 | none (definitional, not an executable check) | metric-name/scope documentation matches code field names | `experiments/claims/README.md`, `docs/api-websocket.md` | n/a — documentation consistency, not tested |
| RS-6 | `full-duration` benchmark batch (20 episodes/mode) | mean RMS altitude/reference error, mean `safety_time_fraction` per `controller_mode` | `experiments/results/raw/full-duration/**`, `experiments/results/tables/full_duration_aggregate.csv`, `experiments/results/tables/full_duration_winner_counts.csv` | one recorded batch; figures `fig03`–`fig06`, `fig10`–`fig12` under `experiments/results/figures/` |
| RS-7 | `coarse-20x50` batch (1000 episodes/mode) + one `fight_mode` 60 s run | mean RMS reference error (coarse); RMS altitude error (fight-mode) per `controller_mode` | `experiments/results/raw/coarse-20x50/**`, `experiments/results/tables/coarse_20x50_aggregate.csv`, `experiments/results/tables/fight_mode_60s_smoke.csv` | figures `fig08_coarse_sweep.png`, `fig09_fight_smoke.png` |

## Unverified or Partially Supported Items

| ID | Missing Evidence or Limitation | Decisive Next Test | Owner Decision Required |
|---|---|---|---|
| RS-6 | The comparative ranking (SHARQ-HJB best on tracking, Q-residual best on safety fraction) comes from one stored benchmark batch (`full-duration`, 20 episodes/mode) with no automated regression test asserting the direction of these comparisons across future code changes; a re-run is not scheduled or CI-wired | Add a bounded CI/nightly job that reruns `experiment_runner.compare_all_methods` on a small scenario subset and asserts the recorded winner-count direction does not regress | Owner decision: whether a comparative-metric regression check is worth the added CI runtime for a companion-code repo (not the paper of record) |
| RS-7 | The fight-mode smoke result is a single run (`seed=777`, `N=1`), not a repeated-seed sample — the SHARQ-HJB-lowest-altitude-RMS finding there has no variance estimate | Extend the fight-mode smoke to multiple seeds, mirroring the coarse sweep's 50-seed design | Owner decision: whether a multi-seed fight-mode batch is worth the added simulation time for a smoke-level check |
| — | Frontend lint/build (`npm run lint`, `npm run build`) and browser smoke (`tests/browser-smoke.spec.ts`) are documented and were run manually for `reproducibility/VALIDATION_2026-05-19.md`, but neither is wired into `.github/workflows/ci.yml` | Add a frontend job to `ci.yml` running `npm run lint`, `npm run build`, and `npm run smoke:browser` | Owner decision: whether Playwright-in-CI cost is justified for a simulation-visualization frontend |
| — | No automatic resume for an interrupted offline benchmark batch (`experiment_runner.run_batch`) — a mid-batch crash leaves partial `experiments/results/raw/` output with no repository-level detection | Add a batch-manifest check that flags an incomplete scenario set before `scripts/sync_results.py` runs | Owner decision: whether partial-batch detection is worth the added tooling given batches are currently short enough to rerun in full |
