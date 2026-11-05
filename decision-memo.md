# Model Decision Memo

## Recommendation

**Keep the traditional model** (the tuned gradient boosting model from
notebook 04) for the Kickstarter success tool. Do not ship the neural network.

## Decision Context

Creators planning a Kickstarter campaign see their chance of being fully
funded before launch and use a what-if tool to try different goals, lengths
and launch dates. The tool gives advice; the creator decides. Hard
constraints: launch-time information only, CPU hardware, under 100 ms per
prediction, and logical what-if advice (a higher goal must never raise the
chance).

## Strongest Evidence

- On the same 66,335 test campaigns the network is **worse**, not better:
  ROC-AUC 0.75895 on average over 5 seeds versus 0.76523 for gradient
  boosting (−0.0063). Every seed is worse. The plan required **+0.005 better**
  ([report: Predictive Results](model-comparison-report.md#predictive-results)).
- The network gives **illogical what-if advice** for 11.6% of campaigns: a
  higher goal raises its predicted chance by up to 4.4 percentage points. The
  gradient boosting model is constrained and never does this
  ([report: Error Analysis](model-comparison-report.md#error-analysis)).

## Most Important Trade-Off

Keeping gradient boosting gives up a model that is 5× faster on large batches
and has a smaller file. The tool scores one campaign at a time, where gradient
boosting is already fast enough (12.5 ms), so nothing the product needs is
lost. In return the team keeps better accuracy, slightly better probabilities
(Brier 0.1916 vs 0.1941), guaranteed logical advice, a 23-second retrain and no
TensorFlow dependency.

## Contradicting Evidence and Uncertainty

- The network is slightly better for Crafts, Fashion, Comics and Photography
  and for goals under $1,000; these gains are small and not tested for
  stability.
- The network reached the 30-epoch cap in most runs and might improve a little
  with longer training. It would need about +0.011 ROC-AUC to change the
  decision, which is unlikely.
- Both models were evaluated on a random split; real future performance may be
  lower for both.
- **No ML** (the goal rule) remains far weaker: ROC-AUC 0.629 and no
  probability to show. It is not a credible replacement, but it is a useful
  fallback message ("smaller goals succeed more often") if the model is
  unavailable.

## Operating Conditions

- Model: `HistGradientBoostingClassifier` as in notebook 04, monotonic in
  `log_goal`; show the probability, and use threshold 0.318 only where a
  yes/no label is needed.
- Before release, **re-run notebook 04** so that `models/kickstarter_hgb.joblib`
  and `data/X_test.csv` match the reported model. The current files on disk
  are stale (no monotonic constraint, old `duration_days`).
- Monitor: share of campaigns per category, goal distribution, and
  calibration (predicted vs. actual success rate) on new campaigns.
- Retrain at least yearly; the data ends in early 2018 and success rates
  changed by year.

## Flip Conditions

The recommendation should be reviewed if:

- a neural network (for example with more epochs or a monotonic architecture)
  beats gradient boosting by at least +0.005 ROC-AUC in every seed **and** gives
  logical goal advice;
- the product adds unstructured inputs such as the campaign description,
  images or video, where networks have a real advantage;
- the product moves to scoring millions of campaigns in batch, where the
  network's throughput would matter; or
- the success rate on new campaigns drifts and gradient boosting's
  calibration degrades while a challenger's does not.

## Next Decision Point

The project team reviews this decision when new Kickstarter data (after 2018)
becomes available, or when campaign text is added as a feature. The review
should repeat the protocol in `model-comparison-plan.md` on a **time-based**
split.
