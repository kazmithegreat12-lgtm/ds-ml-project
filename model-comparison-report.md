# Model Comparison Report

All numbers come from one run of
[`05_neural-network-challenger.ipynb`](05_neural-network-challenger.ipynb) on
2026-11-05. Values are unrounded where they decide something.

## Scope and Protocol

The comparison follows [`model-comparison-plan.md`](model-comparison-plan.md),
which was written before the network was trained. Deviations:

1. **The historical baseline had to be rebuilt.** The saved file
   `models/kickstarter_hgb.joblib` did not match the committed notebook 04: it
   had no monotonic goal constraint, different settings (300 trees, 127
   leaves, `min_samples_leaf=50`) and was trained on an older `duration_days`
   feature (the 30-day default counted as 29 days; 66,330 of 66,335 test rows
   differ). The plan's "re-run gives 0.7663" was measured with that stale
   file and the equally stale `data/X_test.csv`. We rebuilt the model exactly
   as notebook 04 documents it (pipeline, monotonic constraint, best search
   settings, out-of-fold max-F1 threshold). It reproduces the reported result:
   **ROC-AUC 0.76523, threshold 0.318**. This strengthens the evidence: the
   baseline is now the model that was actually reported.
2. **Early stopping almost never triggered.** 11 of 12 trials and 4 of 5 seeds
   reached the 30-epoch cap. The network might still improve slightly with
   more epochs (see limitations). The cap was part of the frozen budget and
   was not changed after seeing results.

No other deviation. The test set was used only for the final evaluation.

## Reproducibility Record

| Item | Historical ML | Optional ML challenger | Neural network |
|---|---|---|---|
| Code location | `04_kickstarter-success.ipynb`; rebuilt in `05_…ipynb` | Not applicable: baseline was already tuned | `05_neural-network-challenger.ipynb` |
| Data and split | `ks-projects-201801.csv` (SHA-256 `4846177c…87e3cb`), 331,675 rows, stratified 80/20, `random_state=42` | — | Same split, test indices asserted equal to `data/X_test.csv`; training further split 85/15 (225,539 fit / 39,801 validation) |
| Preprocessing | Ordinal codes for 4 categoricals (native categorical support), numbers unchanged | — | Embeddings for 4 categoricals, `StandardScaler` on 11 numbers; both fitted on the fit set only |
| Main hyperparameters | `learning_rate=0.0334`, `max_iter=1000` (early stopping), `max_leaf_nodes=63`, `min_samples_leaf=20`, `l2=0`, monotonic `log_goal=-1` | — | 3 dense layers × 256 ReLU, dropout 0.1, L2 1e-4, embedding size 16, Adam lr 0.00058, batch 1,024, ≤30 epochs, early stopping patience 3 on val AUC; 153,923 weights |
| Seeds | `random_state=42` (deterministic) | — | Search seed 42; final seeds 0, 1, 2, 3, 4 |
| Software and hardware | Python 3.13.15, scikit-learn 1.9.0; Intel i3-1115G4 (2 cores / 4 threads), 7.6 GB RAM, CPU only | — | Same + TensorFlow 2.21.0 / Keras 3.15.1, CPU only |
| Tuning budget | 12 configurations × 3-fold CV | — | 12 configurations × 1 validation split |

## Predictive Results

Test set: 66,335 campaigns, 40.4% successful. Thresholds were chosen on
out-of-fold (baseline) or validation (network) data.

| Evidence | Historical ML | Optional ML challenger | Neural network (5 seeds) |
|---|---|---|---|
| Primary metric: ROC-AUC | **0.76523** | — | mean **0.75895**, std 0.00073, range 0.75768–0.75952 |
| Secondary metrics | F1 0.66134, precision 0.54917, recall 0.83110, accuracy 0.65623 | — | F1 0.65793, precision 0.54533, recall 0.82917, accuracy 0.65178 (means) |
| Variability | Deterministic for fixed seed; CV std 0.002 in notebook 04 | — | ROC-AUC std 0.00073; threshold varies 0.305–0.344 between seeds |
| Practical difference (≥ +0.005) | Reference | — | **Not reached.** Mean −0.00628 (−0.82% relative), worst seed −0.00755. Even the 5-seed average (an ensemble) is worse: 95% paired bootstrap interval −0.00479 to −0.00213 |
| Calibration | Brier 0.19159, log loss 0.56068 | — | Brier 0.19405, log loss 0.56662 (slightly worse; both curves close to the diagonal) |
| Costly errors (seed 0 for network) | 18,279 false hope; 1,832 confident false hope (p ≥ 0.7); 4,525 missed successes | — | 18,535 false hope; 1,970 confident false hope; 4,593 missed successes |
| Subgroups / conditions | — | — | Worse in 11 of 15 main categories (largest gap Journalism −0.027, Publishing −0.016); better in Crafts (+0.011), Fashion (+0.006); worse in every launch year and in 6 of 7 goal buckets |

No-ML goal rule for reference: ROC-AUC 0.62908, F1 0.54484, no probability.

## System Results

| Evidence | Historical ML | Optional ML challenger | Neural network |
|---|---|---|---|
| Final training time | 23 s | — | 131 s (seed 0; 108–236 s across seeds) |
| Total tuning effort | 36 fits, not timed in notebook 04 (≈14 min estimated from one fit) | — | 12.1 min search (359 epochs) + 10.2 min for 5 seeds, measured |
| Median / p95 single-campaign latency | 12.5 / 15.6 ms | — | 16.8 / 22.0 ms |
| Batch latency (66,335 rows) | 2,287 ms median | — | 431 ms median |
| Throughput | ≈ 29,000 rows/s | — | ≈ 154,000 rows/s |
| Artifact size | 3.1 MB | — | 1.9 MB (model + preprocessing) |
| Hardware / memory | CPU only, scikit-learn | — | CPU only, TensorFlow (≈ 500 MB dependency) |

Both models meet the 100 ms single-prediction constraint.

## Required Comparison Table

| Dimension | Historical ML | Optional ML challenger | Neural network | No ML / rule |
|---|---|---|---|---|
| Primary metric (ROC-AUC) | 0.76523 | Not applicable: baseline already tuned with 12 trials | 0.75895 ± 0.00073 | 0.62908 |
| Practical difference reached? | Reference | — | No (−0.00628) | No (−0.136) |
| Costly error profile | 1,832 confident false hope | — | 1,970 confident false hope | Cannot express confidence; 61% recall |
| Calibration / probability quality | Brier 0.19159 | — | Brier 0.19405 | Not applicable: no probability |
| Stability across runs | Deterministic | — | Very stable AUC; threshold varies 0.305–0.344 | Deterministic |
| Training and tuning effort | 23 s fit; ≈14 min search (estimated) | — | 131 s fit; 22 min search + seeds (measured) | None |
| Inference latency and throughput | 12.5 ms single; 29k rows/s | — | 16.8 ms single; 154k rows/s | Negligible |
| Model size and hardware | 3.1 MB, CPU | — | 1.9 MB, CPU, TensorFlow | None |
| Interpretability and auditability | Good; monotonic goal guaranteed | — | Weak; goal effect not monotonic for 11.6% | Fully transparent |
| Maintenance and monitoring | scikit-learn only | — | Extra framework, seed repetition, vocabularies | Trivial |
| Product value and risk | Best ranking and logical advice | — | Lower ranking, illogical advice risk | Weak advice, no probability |
| Evidence confidence | High (reproduced) | — | High (5 seeds) | High |

## Error Analysis

- Both models make **the same kind of mistakes**. The most confident false-hope
  cases of the network are small Tabletop Games campaigns (goals of $20–$500)
  that failed; the baseline gave them similar probabilities (0.90–0.98). These
  failures are not visible in launch-time features, so neither model can fix
  them.
- At their chosen thresholds both models flag about 83% of successful
  campaigns; about 45% of their "likely success" calls are false hope.
- **What-if logic.** The baseline never raises the chance when the goal goes
  up (monotonic constraint). The network does so for **11.6%** of 1,000 sampled
  campaigns, by up to **4.4 percentage points**. In the creator tool this would
  mean advice such as "raise your goal to improve your odds", which is
  illogical. This violates a hard constraint from the plan.
- **Contradicting evidence.** The network is better in Crafts, Fashion,
  Comics and Photography and for goals under $1,000, is 5× faster on large
  batches and its file is smaller. None of these addresses the product's main
  need, and the subgroup gains are small and come from single runs per
  segment.

## Operating and Governance Assessment

| Aspect | Historical ML | Neural network |
|---|---|---|
| Interpretability | Feature importances and partial dependence available; monotonic goal effect guaranteed | Needs post-hoc tools (e.g. SHAP); no guaranteed goal effect |
| Auditability | Single deterministic scikit-learn pipeline | Results vary by seed; threshold moves by up to 0.04 between seeds |
| Retraining | One 23 s fit, no extra preprocessing | Scaler + vocabularies + 2 min fit; must repeat seeds to trust a new model |
| Monitoring | Category drift, goal distribution, calibration | Same, plus unknown-category rate (embeddings) |
| Skills and dependencies | scikit-learn only | TensorFlow added (large dependency, no GPU support on native Windows) |
| Affected groups / oversight | Advisory tool, creator decides | Same |

## Limitations and Missing Evidence

- Random split, not time split: both scores may be optimistic for future
  campaigns; the comparison between them is still fair.
- The network hit the 30-epoch cap in most runs; a longer run might gain a
  little. It would need about +0.011 ROC-AUC to reach the practical
  difference, almost as much as the whole spread between the worst and best of
  the 12 tuning trials (0.015), so a flip is unlikely but not ruled out.
- The network was tuned on one validation split, the baseline with 3-fold CV.
- Latency was measured on one laptop with 20 repetitions.
- Subgroup results for the network use the seed average; small segments
  (e.g. Journalism, 837 campaigns) are noisy.
- A monotonic network (e.g. constrained layers or lattice models) was not
  tested.

## Traceable Findings

| Finding | Evidence location | Interpretation | Confidence | Open question |
|---|---|---|---|---|
| Baseline file was stale; rebuilt model reproduces 0.765 | Notebook 05, "Recover the Historical Baseline" | Reported result is credible; saved artifacts must be regenerated | High | Re-run notebook 04 to refresh `models/` and `data/X_test.csv` |
| Network is 0.0063 ROC-AUC worse | "Did the Network Clear the Bar?" | Practical difference not reached; difference is consistent across seeds | High | Would 60+ epochs close the gap? |
| Network breaks goal monotonicity for 11.6% of campaigns | "Logical What-If Advice" | Violates a hard product constraint | High | Could a constrained network fix it at what cost? |
| Baseline is slightly better calibrated | "Probability Quality" | Probabilities shown to creators are more trustworthy | Medium (small gap) | — |
| Network is 5× faster in batch, slightly slower per row | "System Trade-Offs" | Irrelevant for an interactive single-campaign tool | Medium (one machine) | — |
