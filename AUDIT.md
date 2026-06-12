# Audit of the Previous Analysis

This document records every defect found in the previous version of this repository (notebook + README), why each was wrong, the corrected result, and where the fix lives in the current `ab_testing.ipynb`. All "corrected" numbers were recomputed from the raw CSVs on the 29 paired days (2019-08-05 dropped from both groups) and match the executed notebook output.

**The two headline reversals:**

1. The "sanity check" that reported z = 1.48, p = 0.14 and concluded the impression split was consistent with valid randomization was arithmetically broken. The correct test gives **z ≈ 458, p ≈ 0** — the opposite conclusion.
2. The Bayesian section's "P(Test > Control) = 1.0000" claims — including an add-to-cart lift of "(10.9%, 14.4%)" — were artifacts of ignoring 11×–125× day-level overdispersion. Corrected: **the add-to-cart lift is not significant** (paired p = 0.078; day-level posterior P ≈ 0.96; p = 0.26 when two anomalous days are excluded).

## A. Broken statistics — frequentist section

| # | What was claimed | Why it was wrong | Corrected result | Fix location |
|---|---|---|---|---|
| 1 | Sanity-check z-test on impressions: z = 1.48, p = 0.14 → "randomization process … was successful". | Unit mixing: the observed proportion was the control share of ~5.4M *impressions* (~0.59), but the standard error was computed from the number of *days* (`sqrt(0.25/60)`). Denominator and numerator described different random variables. Deeper problem: impressions cannot validate randomization when the two campaigns have different spend and delivery — they are an outcome, not a diagnostic. | Correct binomial SE uses total impressions (5.3M on 29 days): **z ≈ 458, p ≈ 0**. Control served 60% of impressions. Reinterpreted as an *exposure comparability check*: the groups are **not** comparable in exposure, so analysis must use rates. | Notebook "Exposure Comparability Check" cells |
| 2 | Power analysis: required binomial sample sizes computed with MDE = 0.01, then compared against sums like total clicks → "sample size is sufficient". | Category error: required sample sizes are *trials*, but the comparison used *successes* (clicks, add-to-carts, purchases). An absolute MDE of 0.01 is incoherent across metrics (>150% relative for purchases at baseline ~0.006). "Reach/Impressions = 0.81" was fed in as if it were a conversion rate. And the whole binomial calculation was irrelevant to the day-level analysis actually performed (n = 29 days/group). | Replaced with a closed-form **paired-t minimum-detectable-effect** calculation: this design can only detect ~**+94% to +152% relative** effects at 80% power. `statsmodels` dependency removed. | Notebook "Minimum Detectable Effect" cells |
| 3 | 95% CIs on raw daily counts using z = 1.96 and unpaired pooled SEs. | n = 30/group calls for t-quantiles, not z; the two campaigns share the same dates, so an unpaired analysis discards the pairing structure; and raw daily counts conflate per-reach effectiveness with audience size. | Primary analysis is **paired** day-level rate comparison (t quantile, df = 28). Corrected paired volume CIs kept as a secondary analysis: clicks diff (−230, +1,665) p = 0.13; add-to-cart (−628, −214) p = 0.0003; purchases (−110, +90) p = 0.84; reach (−50,338, −23,241) p < 0.0001. | Notebook primary/secondary analysis cells |
| 4 | Missing control day 2019-08-05 median-imputed. | Fabricates a day of data (~3.3% of the control sample): every downstream control total was inflated by the medians (+91,579 Reach, +501 purchases, +116,639 impressions, etc.), including the Bayesian totals. | **2019-08-05 dropped from both groups** → 29 paired days, zero imputation. | Notebook data-cleaning cell (with assertions) |

## B. Overconfident Bayesian section

| # | What was claimed | Why it was wrong | Corrected result | Fix location |
|---|---|---|---|---|
| 5 | Pooled Beta-Binomial: "P(Test > Control) = 1.0000" for all three rate metrics; add-to-cart lift 95% interval "(10.9%, 14.4%)". | Pooling 30 days into one binomial (n ≈ 2.7M/1.6M "trials") assumes one constant rate and independent trials. The data are massively overdispersed: empirical day-level standard errors are **11×–125×** the binomial SEs. The resulting HDIs were one to two orders of magnitude too tight. | Day-level conjugate model on paired daily rate differences: Clicks P ≈ 0.999, Purchases P ≈ 0.998, **AddToCart P ≈ 0.96 with a credible interval crossing zero (−7%, +125%) — not a reliable finding**. Naive model retained only as an explicitly labeled failure-mode illustration. | Notebook Bayesian section (overdispersion diagnostic + day-level model) |
| 6 | "Reach (unique users who saw the ad)" used as a per-user Bernoulli denominator. | Reach is a *daily* figure; summing across days double-counts people reached on multiple days, so summed Reach is person-days, not people. Clicks are also not 0/1 per-person outcomes. | Per-reach rates reframed as **relative efficiency measures**, not per-person conversion probabilities; caveat stated wherever rates are used. | Notebook audit-findings and design cells |
| 7 | The frequentist/Bayesian disagreement presented as a paradigm difference ("they are answering different questions" — frequentist counts vs. Bayesian rates). | False dichotomy: the disagreement came from the **estimand** (raw daily counts vs. rates per reach), not the inference paradigm. Either paradigm can analyze either estimand. | With the estimand fixed, the paradigms agree (e.g. Purchases/Reach: paired p = 0.0045 ↔ posterior P ≈ 0.998). Reconciliation rewritten accordingly. | Notebook "Reconciling…" cell; README |
| 8 | Probabilities reported as exactly "1.0000" from 100,000 Monte Carlo samples. | A finite-sample Monte Carlo estimate can never establish a probability of exactly 1; the most one can claim is "> 0.9999". | All displayed probabilities capped at "> 0.9999" / "< 0.0001". | `fmt_prob` helper in the notebook |

## C. Data-quality issues (previously ignored)

| # | Finding | Status in old version | Handling now |
|---|---|---|---|
| 9 | Control 2019-08-30: **Purchases (670) > Add to Cart (442)** — hard funnel violation. | Never checked. | Flagged by the programmatic audit; purchase-based conclusions carry an explicit tracking caveat. |
| 10 | Add to Cart > View Content on 7 control days and 1 test day; control 2019-08-10 has Searches (2,475) > Website Clicks (2,277). | Never checked. | Flagged by the audit; measurement-error caveat applied to both groups. |
| 11 | Test 2019-08-12 and 2019-08-19: Reach collapses to 10,598 / 10,698 (vs. ~44k median; 08-12 Reach/Impressions = 0.085 vs. ~0.77 median), producing per-reach rates up to ~10× the control mean. | Never checked; these days silently drove the rate results. | Flagged by the audit; **every rate conclusion re-checked in a sensitivity analysis excluding these days** (clicks and purchases survive; add-to-cart does not). |
| 12 | Control 2019-08-05 missing all metrics. | Median-imputed (see finding 4). | Dropped from both groups. |

## D. Logical fallacies in prose

| # | What was claimed | Why it was wrong | Fix |
|---|---|---|---|
| 13 | "We know our data is logical because the test group spend is greater than the control group spend…" | Circular reasoning — relative spend says nothing about data validity. The actual validity checks (funnel consistency) fail in both groups. | Removed; replaced by the programmatic data-quality audit. |
| 14 | "We don't need Bonferroni correction … because that is used mainly for auto-detection." | Garbled and wrong: multiple-comparison correction exists precisely for situations like testing 5 metrics at α = 0.05 (family-wise false-positive rate ~23% with no correction). | Three pre-designated primary metrics with **Holm** step-down correction. |
| 15 | High p-value (0.14) treated as *confirming* valid randomization. | Absence of evidence taken as evidence of absence — on top of the test itself being broken (finding 1). | Corrected test rejects equality overwhelmingly; interpretation rewritten; the notebook's MDE section makes explicit how weak non-significant results are in this design. |
| 16 | README results table headed "95% Confidence Interval" with a p-value in the impressions row. | Mixed quantities in one column. | README results tables rebuilt with consistent, labeled columns. |

## E. README / repository hygiene

| # | Issue | Fix |
|---|---|---|
| 17 | README referenced a `data/` directory that does not exist (CSVs are at the repository root). | File list corrected. |
| 18 | `statsmodels` was imported by the notebook but absent from the README dependency list and install line. | `statsmodels` removed from the notebook entirely (MDE computed in closed form with scipy); dependency list now matches actual imports. |
| 19 | README claims contradicted by this audit: "✅ Randomization valid", the "P(Test > Control) = 1.0000" table, "1.6M vs. 2.7M unique users". | README rewritten: corrected exposure-check result, corrected Bayesian table (capped probabilities, honest add-to-cart verdict), Reach totals described as person-days. |

## Corrected ground truth (29 paired days, executed notebook output)

| Metric (per Reach) | Control | Test | Paired t p | Holm-adj p | Sensitivity p (excl. 08-12, 08-19) | Day-level posterior P(Test > Control) |
|---|---|---|---|---|---|---|
| Clicks/Reach | 0.0645 | 0.1780 | **0.0023** | 0.0069 | 0.0001 | ≈ 0.999 |
| AddToCart/Reach | 0.0159 | 0.0253 | 0.0780 (NS) | 0.0780 | 0.2556 | ≈ 0.96 |
| Purchases/Reach | 0.0063 | 0.0143 | **0.0045** | 0.0090 | 0.0025 | ≈ 0.998 |

Cost (Spend column was unused in the old analysis): control $66,818 → 15,161 purchases (**$4.41/purchase**); test $74,595 → 14,869 purchases (**$5.02/purchase**, ~14% worse; daily paired p = 0.15). Cost per click ≈ $0.43 in both groups. Exposure check: z ≈ 458, p ≈ 0 (control served 60% of 5.3M impressions).

**Bottom line replacing both old recommendations:** the test campaign converts better per unit of reach on clicks and purchases, but delivered no more purchases at higher spend; the add-to-cart lift is unproven; and two anomalous delivery days plus funnel violations make all conclusions tentative. Recommendation: don't scale yet — fix delivery targeting and event tracking, then re-test with user-level instrumentation.
