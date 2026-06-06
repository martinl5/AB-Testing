# A/B Test Analysis of Marketing Campaigns

## Overview

This project aims to analyze the effectiveness of a new marketing campaign (Test) compared to the existing one (Control). The A/B test focuses on evaluating key performance metrics such as impressions, clicks, searches, and purchases, using statistical methods to determine if the Test campaign leads to any significant improvement in website engagement.

The analysis includes data exploration, statistical testing, and recommendations based on the results of the A/B test.

## A Note on Analytical Approach

As a data scientist, I tend to think in Bayesian terms. Rather than asking "is the p-value below 0.05?", I find it more useful to ask "what is the probability this change is actually an improvement, and by how much?" This framing maps more naturally to real business decisions, where we care about expected outcomes and uncertainty — not just a binary reject/fail-to-reject verdict.

This project reflects that mindset. The frequentist analysis (confidence intervals, z-tests) is included because it is the industry standard and provides an important sanity-check lens. But the Bayesian section is where I find the real insight: posterior distributions, credible intervals, and P(Test > Control) tell a richer story about what the data actually support.

## Project Structure

- **A/B Test Notebook**: The main analysis is conducted in this Jupyter Notebook (`ab_testing.ipynb`), which walks through each step of the A/B test, from data preparation to final recommendations.
- **Data**: The data used includes key metrics like impressions, clicks, searches, and purchases for both the control and test groups.

## Key Components

### 1. **Experiment Design**
The A/B test is structured to compare two marketing campaigns:
- **Control Campaign**: The existing marketing strategy.
- **Test Campaign**: A new campaign designed to improve engagement metrics.
  
The goal is to assess whether the Test campaign leads to significant improvements in website engagement metrics.

### 2. **Sanity Checks (Invariant Metrics)**
Before analyzing the key metrics, we run **sanity checks** to ensure that the A/B test was properly randomized. These checks focus on **invariant metrics** that are not expected to change between the control and test groups:
- **Impressions**: The total number of impressions should be randomly split between the control and test groups.

We perform a **z-test** for proportions to verify that the split of impressions follows the expected 50-50 distribution. The null hypothesis (H₀) is that there is no significant difference between the control and test groups in terms of impressions.

### 3. **Exploratory Data Analysis (EDA)**
We explore the data to visualize the distribution of key metrics across the control and test groups. Key steps include:
- **Histograms**: Used to visualize the distribution of numerical features like clicks, searches, and purchases across both groups.
- **Time Series Analysis**: Plots showing how key metrics evolve over time, allowing us to detect trends or changes in behavior during the test period.
- **Correlation Matrix**: A heatmap showing the relationships between numerical features across the control and test groups.

### 4. **Key Metrics Evaluation**
The key metrics analyzed in the A/B test are:
- **Impressions**
- **Clicks**
- **Searches**
- **Purchases**
- **Add to Cart Rate**

For each metric, we compare the performance between the control and test groups using statistical tests to determine if the differences observed are statistically significant.

### 5. **Statistical Tests**
We conduct the following statistical tests:
- **Z-test for proportions**: Used for the sanity check on impressions to ensure the randomization was successful.
- **T-tests (or other relevant tests)**: These are used to compare the key metrics (e.g., purchases, clicks) between the control and test groups, determining if there are statistically significant differences.
- **Beta-Binomial Bayesian analysis**: Posterior Beta distributions are computed for each metric's conversion rate; P(Test > Control) is estimated via Monte Carlo sampling.

### 6. **Results & Interpretation**
Results are summarized in the **[Results section below](#results)**. Key takeaway: the frequentist and Bayesian analyses tell complementary stories — the frequentist lens finds fewer raw events in the test group, while the Bayesian lens finds higher per-user conversion rates. See the Results section for the full breakdown with numbers.

### 7. **Recommendation**
Based on the statistical analysis, the conclusion is **not to launch the new Test campaign** due to its negative impact on key metrics, particularly the Add to Cart rate. The Test campaign also increases marketing spend without a proportional improvement in purchases or other crucial metrics, making it less cost-effective.

### 8. **Suggested Next Steps**
- **Refine the Campaign**: Investigate the reasons behind the drop in Add to Cart rate and consider refining the campaign strategy to improve user experience at this crucial stage.
- **Run Further Tests**: Conduct smaller-scale A/B tests on different variations of the campaign to identify potential improvements.
- **Optimize for Cost**: Focus on improving the cost-efficiency of the marketing campaign by ensuring that increases in spend result in meaningful improvements in metrics like purchases.

### 9. **Bayesian A/B Testing**
A complementary Bayesian analysis using the **Beta-Binomial conjugate model** evaluates each metric as a per-user conversion rate (successes ÷ Reach). Key outputs include:
- **Posterior distributions** for control and test conversion rates
- **95% Highest Density Intervals (HDI)** — the Bayesian equivalent of confidence intervals, with a direct probability interpretation
- **P(Test > Control)** — the posterior probability that the test campaign's true rate exceeds the control's, estimated via Monte Carlo simulation (100,000 samples)
- **Expected lift distribution** — the posterior distribution of the relative improvement in conversion rate

The Bayesian section also reconciles the apparent disagreement with the frequentist results: while the frequentist CIs compared raw daily event counts (finding fewer events in the test group), the Bayesian analysis normalizes by Reach and finds the test campaign converts users at credibly higher rates. Both perspectives are valid but answer different questions.

## Results

### Frequentist Analysis (raw daily metric differences)

| Metric | 95% Confidence Interval | Conclusion |
|---|---|---|
| Impressions (sanity check) | p = 0.14 | ✅ Randomization valid |
| Add to Cart | (−609.5, −230.0) daily events | ❌ Significantly fewer events |
| Purchases | (−100.5, +98.9) daily events | ⚠️ Inconclusive |
| Website Clicks | (−154.5, +1584.1) daily events | ⚠️ Inconclusive |
| Reach | (−48,295, −22,594) daily users | ❌ Significantly fewer users reached |

The frequentist analysis suggested **do not launch** the test campaign: it showed fewer add-to-cart events and reduced reach, with no statistically significant gains in purchases or clicks.

### Bayesian Analysis (per-user conversion rates, denominator = Reach)

| Metric | P(Test > Control) | Expected Lift | 95% Credible Interval |
|---|:---:|:---:|---|
| Add to Cart | **1.0000** | +12.6% | (10.9%, 14.4%) |
| Purchases | **1.0000** | +66.0% | (62.4%, 69.8%) |
| Website Clicks | **1.0000** | +88.6% | (87.4%, 89.8%) |

When normalized by the number of unique users each campaign actually reached, the test campaign converts users at substantially higher rates across every metric.

### Reconciling the Two Perspectives

The apparent contradiction arises because the test campaign reached ~40% fewer unique users (1.6M vs. 2.7M). This compressed raw event counts — not because the campaign was less effective per user, but because it exposed fewer people. The Bayesian analysis controls for this by measuring conversion *rate* rather than absolute volume.

> **Bottom line**: The test campaign is more efficient at converting users it reaches. Whether to launch depends on whether the business can address the targeting inefficiency that led to lower Reach.

## How to Use This Notebook

1. **Install Dependencies**: Make sure you have the required Python packages installed:
   ```bash
   pip install pandas numpy scipy matplotlib seaborn
   ```

2. **Load Data**: Ensure that the control and test group datasets are loaded in the correct format. The dataset should contain key metrics such as impressions, clicks, and purchases.

3. **Run the Notebook**: Open the `ab_testing.ipynb` notebook and run the cells sequentially to reproduce the analysis. The notebook will walk you through the following:
   - Data exploration and visualization
   - Statistical tests to compare control and test groups
   - Final recommendations based on the results

## Files
- **ab_testing.ipynb**: The Jupyter Notebook containing all the analysis and visualizations.
- **data/**: Directory where control and test datasets are stored (replace this with actual dataset filenames if provided).

## Dependencies
- Python 3.6 or higher
- Libraries:
  - pandas
  - numpy
  - scipy
  - matplotlib
  - seaborn

## Contact
For questions or issues regarding this analysis, feel free to reach out to the author at [martin.lim511@gmail.com] or submit an issue on this repository.
