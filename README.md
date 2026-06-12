# A/B-Style Comparison of Two Marketing Campaigns

## Overview

This project compares the performance of two marketing campaigns — an existing **Control** campaign and a new **Test** campaign — that ran over the same 30 days in August 2019 (Kaggle dataset). The analysis covers data-quality auditing, exploratory analysis, frequentist paired tests, a cost-efficiency analysis, and a Bayesian day-level model.

**Important framing:** the data are *daily aggregates per campaign* (spend, impressions, reach, clicks, searches, content views, add-to-carts, purchases) — there is no user-level assignment. This is therefore a **quasi-experimental comparison of two campaigns**, not a user-randomized A/B test, and the unit of analysis is the **campaign-day** (29 paired days after cleaning).

> ⚠️ **This repository was substantially corrected.** An earlier version contained a broken sanity-check z-test, a category-error power analysis, fabricated (median-imputed) data for a missing day, and a Bayesian model whose "P(Test > Control) = 1.0000" conclusions were artifacts of ignoring 11×–125× overdispersion. See **[AUDIT.md](AUDIT.md)** for the full finding-by-finding account. The headline reversals:
>
> - The old "sanity check" reported z = 1.48, p = 0.14 and declared the impression split valid. The correctly computed test gives **z ≈ 458, p ≈ 0** — the campaigns bought *completely different* amounts of exposure (60/40 split of 5.3M impressions).
> - The old Bayesian section reported "P(Test > Control) = 1.0000" for all three rate metrics, including an add-to-cart lift of (10.9%, 14.4%). The corrected day-level analysis finds the **add-to-cart lift is not significant** (paired p = 0.078; posterior P ≈ 0.96; weaker still when two anomalous days are excluded).

## A Note on Analytical Approach

I find Bayesian framing more decision-relevant than binary significance verdicts: "what is the probability this change is an improvement, and by how much?" maps directly to business choices. But this project is also a lesson in why **the model matters more than the paradigm**: a Bayesian model that ignores the data-generating process (here, massive day-to-day overdispersion) produces confident nonsense. The corrected notebook shows the naive model, diagnoses its failure, and replaces it with a day-level model whose conclusions agree with the frequentist analysis on the same estimand.

## Data Quality

The raw data has real problems, all surfaced programmatically in the notebook's audit section:

| Issue | Detail | Handling |
|---|---|---|
| Missing day | Control 2019-08-05 has no metrics | Dropped from **both** groups (no imputation) → 29 paired days |
| Funnel violation | Control 2019-08-30: 670 purchases but only 442 add-to-carts | Kept, flagged — purchase tracking is unreliable |
| Funnel anomalies | Add-to-cart > view-content on 7 control days + 1 test day; searches > clicks on control 2019-08-10 | Kept, flagged |
| Delivery collapse | Test 2019-08-12 and 2019-08-19: Reach ≈ 10.6k vs a 44k median (08-12 has Reach/Impressions = 0.085); per-reach rates spike up to ~10× | Kept, but every conclusion re-checked in a sensitivity analysis excluding these days |
| Reach semantics | Reach is a *daily* figure; summing it double-counts people seen on multiple days | Rates treated as relative efficiency measures, not per-person probabilities |

## Analysis Design

- **Unit of analysis:** campaign-day, **paired by date** (both campaigns share the same 29 dates).
- **Three pre-designated primary metrics** (daily rates, denominator = daily Reach): Clicks/Reach, AddToCart/Reach, Purchases/Reach.
- **Multiplicity:** Holm step-down correction across the three primary tests.
- **Robustness:** Wilcoxon signed-rank and sign-flip permutation tests alongside each paired t-test.
- **Sensitivity:** all tests re-run excluding the two anomalous test-delivery days.
- **Design sensitivity:** a closed-form paired-t MDE calculation shows 29 daily aggregates can only detect ~94–152% *relative* effects — this design cannot see small lifts.
- **Secondary:** paired comparisons of raw daily volumes, and a cost-efficiency analysis (the Spend column was unused in the original analysis).

## Results

### Exposure comparability (impressions)

Control served 3.18M impressions vs. test's 2.12M (60/40 split): **z ≈ 458, p ≈ 0**. The groups are *not* comparable in exposure, so raw counts cannot be compared directly — all primary analysis is on rates.

### Primary metrics — paired day-level rates (n = 29 days)

| Metric (per Reach) | Control | Test | 95% CI of diff | p (paired t) | p (Holm) | p excl. anomalous days | Verdict |
|---|---|---|---|---|---|---|---|
| Clicks/Reach | 0.064 | 0.178 | (0.044, 0.183) | 0.0023 | 0.0069 | 0.0001 | ✅ Test higher |
| AddToCart/Reach | 0.016 | 0.025 | (−0.001, 0.020) | 0.0780 | 0.0780 | 0.2556 | ⚠️ Not significant |
| Purchases/Reach | 0.006 | 0.014 | (0.003, 0.013) | 0.0045 | 0.0090 | 0.0025 | ✅ Test higher |

Wilcoxon and permutation tests agree with each verdict.

### Volumes and cost (the other half of the story)

| Quantity | Control | Test | Note |
|---|---|---|---|
| Total reach (person-days) | 2.58M | 1.51M | Test reached far fewer people (p < 0.0001) |
| Total purchases | 15,161 | 14,869 | No difference (p = 0.84) |
| Total add-to-carts | 37,700 | 25,490 | Test significantly lower (p = 0.0003) |
| Total spend | $66,818 | $74,595 | Test spent ~12% more |
| **Cost per purchase** | **$4.41** | **$5.02** | ~14% worse for test (daily paired p = 0.15) |
| Cost per click | $0.43 | $0.43 | Identical |

### Bayesian analysis (day-level model)

The naive pooled Beta-Binomial model (kept in the notebook as a failure-mode illustration) claims P(Test > Control) > 0.9999 for all three metrics with implausibly tight intervals. An overdispersion diagnostic shows its standard errors are understated by **11×–125×**. The corrected model — a conjugate Normal–Inverse-Gamma posterior on the paired daily rate differences — gives:

| Metric (per Reach) | P(Test > Control) | Relative lift (mean) | 95% credible interval |
|---|:---:|:---:|---|
| Clicks/Reach | ≈ 0.999 | +176% | (+68%, +284%) |
| AddToCart/Reach | ≈ 0.96 | +59% | (−7%, +125%) |
| Purchases/Reach | ≈ 0.998 | +127% | (+42%, +211%) |

(Monte Carlo probabilities are reported as bounded values, never as exactly 0 or 1.)

### Why the count-based and rate-based results differ

The earlier README framed this as "frequentist vs. Bayesian". That was wrong: the two sections analyzed **different estimands** — raw daily *counts* vs. *rates per reach*. With the estimand fixed, the paradigms agree (e.g. Purchases/Reach: paired p = 0.0045 ↔ posterior P ≈ 0.998). Both descriptions are true simultaneously: the test campaign converted a much smaller delivered audience at much higher rates, ending with the same purchase volume at higher cost.

## Recommendation

**Do not scale the test campaign yet — and do not kill it either. Fix delivery and measurement, then re-test.**

- Per unit of reach, the test campaign clicks and purchases at dramatically higher rates (robust to multiplicity correction, non-parametric tests, and outlier exclusion).
- The add-to-cart lift is unproven.
- As delivered, the test campaign produced **no more purchases at ~12% higher spend** (cost per purchase $5.02 vs. $4.41).
- The rate advantage is entangled with two anomalous delivery days and funnel-tracking violations, so the per-reach superiority may partly reflect a narrower, warmer audience rather than better creative.

Next steps: diagnose why test's delivery collapsed onto a small high-converting audience; fix funnel tracking and capture revenue (for ROAS); re-run with matched targeting/budgets — ideally with user-level instrumentation, since daily aggregates can only detect ~100%+ relative effects.

## How to Use This Notebook

1. **Install dependencies:**
   ```bash
   pip install pandas numpy scipy matplotlib seaborn
   ```
2. **Run the notebook:** open `ab_testing.ipynb` and run all cells top to bottom (random seeds are fixed; results are reproducible).

## Files

- **`ab_testing.ipynb`** — the full analysis: data-quality audit, EDA, design, frequentist + cost analysis, Bayesian analysis.
- **`control_group.csv`**, **`test_group.csv`** — daily aggregates for each campaign (semicolon-separated), at the repository root.
- **`AUDIT.md`** — finding-by-finding audit of the errors in the previous version of this analysis and how each was fixed.

## Dependencies

- Python 3.9+
- pandas, numpy, scipy, matplotlib, seaborn

## Contact

For questions or issues regarding this analysis, reach out at [martin.lim511@gmail.com] or open an issue on this repository.
