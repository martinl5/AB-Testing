# A/B Test Analysis of Marketing Campaigns

## Overview

This project aims to analyze the effectiveness of a new marketing campaign (Test) compared to the existing one (Control). The A/B test focuses on evaluating key performance metrics such as impressions, clicks, searches, and purchases, using statistical methods to determine if the Test campaign leads to any significant improvement in website engagement.

The analysis includes data exploration, statistical testing, and recommendations based on the results of the A/B test.

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

### 6. **Results & Interpretation**
- **Statistical Significance**: The test metrics are evaluated using p-values and confidence intervals to determine if the new campaign had a statistically significant impact on user behavior.
- **Effect on Key Metrics**: The Add to Cart rate showed a significant decrease in the test group, raising concerns about the Test campaign's impact on conversions.
  
If the p-value is less than the threshold (e.g., 0.05), the test results are considered statistically significant. In this case, a statistically significant drop in Add to Cart rates was detected.

### 7. **Recommendation**
Based on the statistical analysis, the conclusion is **not to launch the new Test campaign** due to its negative impact on key metrics, particularly the Add to Cart rate. The Test campaign also increases marketing spend without a proportional improvement in purchases or other crucial metrics, making it less cost-effective.

### 8. **Suggested Next Steps**
- **Refine the Campaign**: Investigate the reasons behind the drop in Add to Cart rate and consider refining the campaign strategy to improve user experience at this crucial stage.
- **Run Further Tests**: Conduct smaller-scale A/B tests on different variations of the campaign to identify potential improvements.
- **Optimize for Cost**: Focus on improving the cost-efficiency of the marketing campaign by ensuring that increases in spend result in meaningful improvements in metrics like purchases.

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
