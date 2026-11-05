# Model Comparison Plan

Completed on 2026-11-05, **before** the neural network was trained or evaluated
on the test set. Any later change to this plan is listed as a deviation in
[`model-comparison-report.md`](model-comparison-report.md).

## Product Decision

- **Decision supported by the model:** a creator planning a Kickstarter
  campaign sees the probability that the campaign will be fully funded, before
  launch, and uses it to adjust goal, category, timing and length.
- **People or process affected:** individual creators (the users), and
  indirectly backers and the platform. Nobody is approved or rejected by the
  model; it is advice.
- **Cost of important errors:**
  - *False hope* (high probability, campaign fails): the creator invests time
    and money in a campaign that pays out nothing (all-or-nothing funding).
    This is the most expensive error for the user.
  - *False discouragement* (low probability, campaign would succeed): a viable
    project is delayed or not launched. Costly, but the creator can still
    decide to launch.
  - Because the product shows a probability, badly **calibrated** probabilities
    are an error in themselves: "60%" must mean about 60%.
- **Hard constraints:**
  - Only information known at launch time (no `pledged`, `backers`, `state`).
  - Must run on ordinary CPU hardware; no GPU is assumed for serving.
  - Single prediction latency must stay below 100 ms on the reference laptop
    (interactive what-if tool).
  - The what-if advice must be logical: a higher goal must never raise the
    predicted chance (the historical model enforces this with a monotonic
    constraint).

## Candidate Systems

| Candidate | Role | Current evidence |
|---|---|---|
| No ML / rule | Simple alternative: "goal at or below the training median ($5,000) is likely to succeed" | Test ROC-AUC 0.629, F1 0.545 (`04_kickstarter-success.ipynb`) |
| Historical traditional ML | Existing baseline: tuned `HistGradientBoostingClassifier` saved in `models/kickstarter_hgb.joblib` | Notebook reported test ROC-AUC 0.765, F1 0.661; re-run on 2026-11-05 gives 0.7663 / 0.6624 |
| Optional traditional ML | Not added | The historical model was already tuned (12-trial randomized search), so a stronger traditional challenger is not needed for a fair comparison |
| Neural network | New challenger: small feed-forward network with category embeddings (Keras) | None yet |

## Frozen Data Protocol

- **Dataset version or hash:** `data/ks-projects-201801.csv` (Kaggle
  "kemical/kickstarter-projects", 2018-01 release). The SHA-256 is printed by
  `05_neural-network-challenger.ipynb`.
- **Target:** `success` = 1 if `state == "successful"`, 0 if `"failed"`; all
  other states removed (331,675 rows).
- **Training observations:** 265,340 rows from the existing stratified 80/20
  split (`random_state=42`).
- **Validation or cross-validation strategy:** the neural network splits the
  training set once more, stratified 85/15 (`random_state=42`): 225,539 rows
  for fitting, 39,801 rows for early stopping, configuration choice and
  threshold choice. (The historical model used 3-fold CV on the training set
  for the same purposes; one validation split is used for the network because
  each network fit is slower.)
- **Untouched test observations:** the same 66,335 rows saved in
  `data/X_test.csv`. The notebook regenerates the split from the raw data and
  asserts that the row indices are identical to that file.
- **Group or time constraints:** none enforced, to match the historical model.
  This is a known limitation: a random split lets the model learn from
  campaigns launched after the test campaigns (see limitations).
- **Leakage checks:** outcome columns are dropped before the split; the
  scaler, category vocabularies, threshold and early stopping are all fitted
  on training or validation data only; the test set is predicted once per
  final model.

## Evaluation Protocol

- **Primary metric:** ROC-AUC on the 66,335 test rows.
- **Secondary metrics:** F1, precision and recall of the success class at a
  threshold chosen on validation data (maximum F1, as for the historical
  model); log loss and Brier score for probability quality; accuracy for
  reference only.
- **Practical difference required:** the network must beat the historical
  model by **at least +0.005 ROC-AUC** (mean over seeds), and the worst seed
  must also beat it. Smaller gains are below what a creator could notice and
  about the size of the historical model's fold-to-fold spread (0.002) plus
  its tuning gain (0.0005), so they would not justify a second, heavier model.
- **Subgroups or conditions:** ROC-AUC by `main_category`, by goal bucket and
  by launch year.
- **Calibration or threshold plan:** reliability curve (10 bins) and Brier
  score for both models on the test set. Thresholds are chosen on validation /
  out-of-fold data only, never on the test set.
- **Repeated-run policy and seeds:** the final network configuration is
  trained with **5 seeds (0, 1, 2, 3, 4)**; mean, standard deviation and range
  are reported. The historical model is deterministic for a fixed
  `random_state` and is evaluated once.

## Resource Protocol

- **Hardware:** Windows 11 laptop, Intel Core i3-1115G4 (2 cores, 4 threads),
  7.6 GB RAM, no usable GPU. Python 3.13, scikit-learn 1.9.0, TensorFlow 2.21.
- **Tuning budget per challenger:** **12 configurations**, sampled at random
  with seed 42, matching the 12 configurations of the historical randomized
  search. Each configuration trains for at most 30 epochs with early stopping
  (patience 3, on validation ROC-AUC).
- **Training-time method:** wall-clock time of one final fit and of the whole
  search, measured with `time.perf_counter()` in the same notebook run.
- **Inference workload, warm-up, and repetitions:** (a) one batch of all
  66,335 test rows and (b) a single row, as in the what-if tool. 3 warm-up
  calls are discarded, then 20 timed repetitions; median and 95th percentile
  are reported.
- **Artifact-size or memory method:** size on disk of the saved model file
  (`.joblib` for the historical model, `.keras` plus preprocessing for the
  network).

## Expected Decision Rule

- **Neural network** if it reaches the practical difference (+0.005 ROC-AUC,
  stable across seeds), is at least as well calibrated, meets the 100 ms
  latency constraint, and its what-if advice is not illogical.
- **Traditional ML (keep the historical model)** if the network does not reach
  the practical difference, or reaches it only by violating a constraint.
- **No ML** if neither model clearly beats the goal rule (ROC-AUC 0.629) or if
  probabilities are too poorly calibrated to show to creators.
- **Further experiment** if the result is close to the threshold and the
  variation across seeds makes the decision uncertain.

## Known Limitations Before Training

- The random split is not a time split, so both models may look slightly
  better than they would on future campaigns.
- The data ends in early 2018; platform behaviour may have changed since.
- The network does not have a built-in monotonic constraint on the goal; this
  must be checked separately.
- One validation split for the network versus 3-fold CV for the historical
  model is not identical tuning, only a comparable budget.
- Latency is measured on one laptop and will differ on a server.
