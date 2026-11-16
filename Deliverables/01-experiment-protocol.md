# 01 — Problem and evaluation protocol

Status: frozen initial plan, 2026-11-16. No experiments have run.

## Objective and data

Predict an engine's remaining useful life (RUL), in cycles, from its observed sensor history. Start with NASA C-MAPSS FD001; FD002/FD003 extend the generalization study later. These are simulated trajectories, not operational aircraft validation.

Source: [NASA dataset repository, entry 6](https://www.nasa.gov/intelligent-systems-division/discovery-and-systems-health/pcoe/pcoe-data-set-repository/).

For a training engine failing at cycle T, the target at cycle t is **y = T - t**. Use actual, uncapped RUL initially. Official test labels describe RUL at each engine's last observed cycle. Failure time and future measurements are never model inputs.

Available inputs: current cycle, three operating settings, and 21 sensor histories through t. Engine ID identifies groups only; it is not a feature. Compare models with and without current cycle. Outputs are a nonnegative point estimate and, later, 80%/90% prediction intervals. Apply the same nonnegative clipping rule during validation and deployment.

## Engine splits and prediction cutoffs

- Sort FD001 training engine IDs, permute with NumPy default_rng(42), then allocate 60 training, 20 validation, and 20 calibration engines. Persist actual IDs and data checksums before training.
- Split engines before generating features or windows. Official test engines remain separate.
- For each validation/calibration engine of length T, draw one integer cutoff uniformly from ceil(0.2*T) through T-1 inclusive. Use separate RNG streams seeded 43 and 44, respectively, traversing sorted IDs. Save cutoffs once and reuse them across models.
- This synthetic prefix distribution is a declared design choice, not a reproduction of NASA's unknown test truncation distribution. Failure time is used to construct evaluation examples and labels only.
- Training can use all eligible causal prefixes, with equal total weight per engine (equivalent engine-balanced sampling for neural training). Never cross engine boundaries.
- For short histories, compute tabular statistics from available observations. Left-pad sequences with the first observed measurement; record this policy and include short histories in evaluation.
- Fit scaling, feature selection, imputation, and operating-condition transforms only on training engines. Use trailing windows, never centered windows or future-based filling.

## Selection and final evaluation

Select configurations by lowest validation endpoint RMSE. Exact ties favor lower model complexity. Calibration data must not influence feature choices, hyperparameters, or model selection.

Freeze comparisons and configurations before accessing official test labels. Refit each selected model family on the 80 training-plus-validation engines; refit its preprocessing on those engines too. Use the 20 untouched calibration engines only to calibrate the already selected predictor. Do not refit after calibration.

For neural refitting, use the epoch count selected by validation early stopping. Compare selected TCN runs with seeds 42, 43, and 44; predesignate seed 42 for deployment rather than choosing the luckiest test run.

Primary official evaluation: one prediction at the final observed cycle per test engine. Trajectory analyses are separate. Test-inspired changes must be labeled exploratory and cannot be presented as independent confirmation.

## Metrics and uncertainty

Report RMSE (primary), MAE, mean signed error, training time, and inference latency on documented hardware. Let d = prediction - actual RUL. The asymmetric score is the sum over engines of:

- exp(-d/13) - 1 when d < 0;
- exp(d/10) - 1 when d >= 0.

Thus overestimating remaining life receives a larger penalty for the same error magnitude. Verify this formula against hand-calculated cases before reporting results.

Report errors by actual RUL [0,30], (30,60], (60,120], and >120 cycles, and observed history <30, 30–99, and >=100 cycles. Include group counts; mark empty groups unavailable. Produce predicted-versus-actual and residual plots.

For paired model comparisons, bootstrap engines with replacement 2,000 times (seed 45) and report 95% percentile intervals for metric differences. Do not bootstrap correlated windows as independent samples. Report neural seed variation separately.

For split-conformal intervals, use one absolute residual per calibration engine and the finite-sample quantile rank ceil((n+1)*(1-alpha)). Intersect intervals with nonnegative RUL. Report empirical coverage, mean width, and coverage breakdowns. Twenty engines give coarse estimates. No claim of conditional, simultaneous-trajectory, or shifted-domain coverage; marginal guarantees require exchangeability of calibration and future examples, which our synthetic cutoffs do not establish for official test endpoints.

## Bounded experiment plan

| Experiments | Initial budget and purpose |
| --- | --- |
| E0–E2: baselines and features | Constant, age-only, Ridge, gradient boosting; latest values versus trailing statistics; age ablation. At most 32 fitted configurations total. |
| E3: temporal model | TCN: history 20/30/50 × two sizes × squared/Huber loss = 12 configurations; maximum 100 epochs each, early-stopping patience 10. |
| E4: target ablation | Train selected classical and TCN configurations with RUL capped at 125; compare against their uncapped counterparts using actual-RUL evaluation. Two additional fits. |
| E5: stability | Two additional seeds for the selected TCN configuration; engine bootstrap comparisons. |
| E6: intervals | Calibrate selected point predictor for 80%/90% intervals; no calibration-driven model search. |
| E7–E8: shift and robustness | FD001 → FD002/FD003 transfer; target-domain references; normalization, noise, and missingness studies. Freeze detailed interventions and budgets before running them. |
| E9: serving | Offline/API parity, latency, and monitoring replay; define acceptance checks before deployment. |

Budgets exclude final refits. Record failed runs; do not silently replace them with an expanded search. Any budget or protocol change requires a dated explanation before the affected experiment. No fixed accuracy target is promised.

## Completion and experiment records

Every run saves configuration, seeds, engine IDs/cutoffs, data checksum, code revision, preprocessing/model artifacts, predictions, metrics, and a short conclusion. Track baseline experiments in MLflow when implemented.

This first deliverable is complete when the target, inputs, splits, cutoffs, metrics, selection rule, and initial budget are documented. Implementation and measured results belong to subsequent stages.
