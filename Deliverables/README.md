# Project deliverables

Work through stages 01–12 in order. **UNDONE** means the stage is pending or in progress; change it to **DONE** only once its deliverables and validation are complete. Record links to the resulting files or experiment reports when completing a stage.

| Stage | Status | Component | Deliverables and completion criteria |
| --- | --- | --- | --- |
| 01 | DONE | Problem and evaluation protocol | [Experiment protocol](01-experiment-protocol.md): target, inputs, engine splits, cutoffs, metrics, selection rule, and bounded experiment plan documented. |
| 02 | UNDONE | Repository and reproducible environment | Installable Python package, dependency setup, configuration files, checks/CI, and reproducible experiment commands recording seeds, data checksums, split IDs, and code version. Repository creation alone does not complete this stage. |
| 03 | UNDONE | Dataset acquisition and validation | NASA C-MAPSS downloader, parser, schema and integrity checks, test-label alignment checks, and dataset inventory. |
| 04 | UNDONE | Exploratory data analysis | Notebook and brief data report covering engine lifetimes, sensor trajectories, near-constant sensors, operating settings, and justified preprocessing choices. |
| 05 | UNDONE | Splits, labels, and features | Engine-disjoint training/validation/calibration splits, documented validation truncation, RUL labels, causal tabular features and sequence windows, training-only preprocessing, and leakage tests. |
| 06 | UNDONE | Baseline models | Constant, age-only, Ridge, and gradient-boosting models; latest-value/temporal-feature and age ablations; reproducible benchmark table and selected classical baseline. |
| 07 | UNDONE | Evaluation and error analysis | Reusable evaluation command producing endpoint RMSE, MAE, asymmetric score, signed errors, RUL/history-length breakdowns, plots, and engine-level comparison uncertainty; verified scoring implementation. |
| 08 | UNDONE | Temporal deep learning | TCN training pipeline with short-history handling, checkpoints and early stopping; bounded history-length/model-size/loss experiments; repeated-seed results and comparison with classical models, training cost, and inference latency. |
| 09 | UNDONE | Prediction intervals | Split-conformal calibration on untouched engines; nominal 80%/90% coverage and interval-width evaluation; breakdowns by RUL/history length; documented calibration sample size and assumption limitations. |
| 10 | UNDONE | Generalization and robustness | FD001-to-FD002/FD003 transfer, target-domain reference models, normalization comparison, sensor-noise/missing-data experiments, and report on accuracy and coverage degradation. FD004 is an optional extension. |
| 11 | UNDONE | Model packaging and serving | Versioned artifact containing schema, preprocessing, model and calibration; validated FastAPI service; offline/API prediction parity checks; Docker image and documented latency measurements. |
| 12 | UNDONE | Tracking, monitoring, and demonstration | Complete MLflow records and model promotion criteria, monitoring replay with shift/false-alarm checks, interactive engine-history dashboard, reproducible deployment demonstration, technical report, and README with measured results. |

## Experiment checklist

Freeze planned comparisons before inspecting official test labels. Tune on development data; keep calibration engines separate. Document any test-inspired follow-up as exploratory. Each experiment should save its configuration, data/split provenance, predictions, metrics, and a short conclusion.

| ID | Status | Experiment |
| --- | --- | --- |
| E0 | UNDONE | Constant and age-only baselines. |
| E1 | UNDONE | Ridge versus gradient boosting. |
| E2 | UNDONE | Latest sensor values versus trailing features; with/without engine age. |
| E3 | UNDONE | TCN versus classical baseline; history lengths 20/30/50, bounded size and loss comparisons. |
| E4 | UNDONE | Raw versus capped training targets, evaluated against the same actual-RUL target. |
| E5 | UNDONE | Repeated runs, including three seeds for the selected TCN, and engine-level paired comparisons. |
| E6 | UNDONE | Prediction interval coverage and width, overall and by RUL/history length. |
| E7 | UNDONE | Cross-subset transfer and target-domain reference models. |
| E8 | UNDONE | Operating-condition normalization, controlled sensor noise, and missing measurements. |
| E9 | UNDONE | API prediction parity, latency, and monitoring replay with false-alarm assessment. |

## Working notes

- Start with FD001 and actual, uncapped RUL in cycles.
- Initial FD001 development allocation: 60 training, 20 validation, and 20 calibration engines; official test engines remain separate.
- Split engines before creating windows. Fit preprocessing only on training data.
- Select configurations on validation data. Refit selected configurations on the 80 training-plus-validation engines before calibrating on the untouched 20 calibration engines.
- Predetermine validation/calibration cutoff sampling. Use one cutoff per calibration engine initially; do not treat overlapping windows as independent calibration samples.
- Primary official benchmark: one prediction at the last observed cycle of each test engine. Report trajectory evaluation separately.
- Introduce MLflow during baseline experiments; stage 12 completes the tracking, registration, monitoring, and demonstration workflow.
- C-MAPSS contains simulated trajectories. Report benchmark evidence and uncertainty limitations without claiming validation on real aircraft.
