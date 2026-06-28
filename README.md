# A/B Testing Basics: Two Marketing Campaigns

A pared-down walkthrough of the **basics of A/B testing**, using two real
marketing campaigns (a **Control** and a **Test** campaign that ran over the
same 30 days in August 2019, [Kaggle dataset](https://www.kaggle.com/datasets/amirmotefaker/ab-testing-dataset/data)).

The notebook does three things:

1. **Load and clean** the daily campaign data.
2. **Compare conversion rates** (Control vs Test) with a simple paired test.
3. **Design an A/B test** — choose a significance level (α), power (1−β) and a
   minimum detectable effect (MDE), then compute the required **sample size per
   variation**. This section mirrors
   [Evan Miller's sample-size calculator](https://www.evanmiller.org/ab-testing/sample-size.html).

## Data

- `control_group.csv`, `test_group.csv` — daily aggregates per campaign
  (semicolon-separated), at the repository root. Columns: spend, impressions,
  reach, website clicks, searches, view-content, add-to-cart, purchases.
- **Cleaning:** 2019-08-05 is dropped from both groups (the control campaign has
  no metrics that day) → **29 paired days** per campaign.
- The data has real quality issues (funnel-tracking violations, two anomalous
  low-reach test days) that are deliberately left in and *flagged*, not fixed —
  see [`AUDIT.md`](AUDIT.md) for the full finding-by-finding record.

## Method

Each row is a *daily aggregate*, and the two campaigns differ in spend and
delivery (the control served ~60% of all impressions), so the groups are **not
comparable in raw exposure**. We therefore compare **rates per unit of reach**,
not raw counts:

- **Clicks/Reach**, **AddToCart/Reach**, **Purchases/Reach**.

Both campaigns share the same 29 dates, so the comparison is **paired** by date
(a paired t-test on the within-date Test − Control differences).

> *Caveat:* summed Reach double-counts people seen on multiple days, so these
> rates are relative efficiency measures, not per-person conversion
> probabilities.

## Results

Paired day-level comparison (n = 29 days):

| Metric (per Reach) | Control | Test  | p (paired t) | Significant (α=0.05) |
|--------------------|:-------:|:-----:|:------------:|:--------------------:|
| Clicks/Reach       | 0.064   | 0.178 | 0.0023       | ✅ yes               |
| AddToCart/Reach    | 0.016   | 0.025 | 0.0780       | ❌ no                |
| Purchases/Reach    | 0.006   | 0.014 | 0.0045       | ✅ yes               |

The other half of the story (raw volumes): the test campaign reached far fewer
people and ended with essentially the **same total purchases** (14,869 vs
15,161) at **~12% higher spend** — cost per purchase **$5.02 vs $4.41**. So the
per-reach rate advantage did **not** translate into more sales.

**Bottom line:** don't scale the test campaign yet. Fix delivery/targeting and
event tracking, then re-test at the user level with an adequate sample size (see
below).

## Designing the test (α, β, MDE, sample size)

Evan Miller's calculator answers "how big does the test need to be?" from four
inputs:

- **Baseline conversion rate** `p` — the current (control) rate.
- **Minimum detectable effect (MDE)** — the smallest lift worth detecting
  (relative or absolute).
- **Significance level α** — false-positive rate (default 0.05).
- **Statistical power 1 − β** — chance of detecting a true effect of size MDE
  (default 0.80).

The notebook's `sample_size()` helper implements the standard two-sided
two-proportion formula (validated against Evan Miller's calculator: baseline
20%, +5% relative lift, α=0.05, power=0.80 → ~25,600 per variation).

Applied to this campaign's low purchase baseline (~0.6%), detecting even a
**+20% relative lift needs ~73,000 users per variation** (a +10% lift needs
~280,000). That is why **29 daily aggregates can only detect very large
(~+100%+) effects** — to measure realistic single/double-digit lifts you need
user-level data at these sample sizes.

## How to run

```bash
pip install pandas numpy scipy matplotlib
jupyter notebook ab_testing.ipynb   # run all cells top to bottom
```

## Files

- **`ab_testing.ipynb`** — the analysis: clean → compare → design (sample size).
- **`control_group.csv`**, **`test_group.csv`** — the daily campaign data.
- **`AUDIT.md`** — detailed audit of the earlier, more complex version of this
  analysis (kept as historical record).

## Dependencies

- Python 3.9+
- pandas, numpy, scipy, matplotlib

## Contact

For questions, reach out at martin.lim511@gmail.com or open an issue.
