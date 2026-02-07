# NYC Taxi Trip Duration Analysis

## Overview
This project analyzes NYC taxi trip data to understand the key drivers of trip duration and to build an interpretable regression baseline that explains how distance and rush hour affect average travel time.

Rather than optimizing purely for prediction accuracy, the project emphasizes model interpretability, assumption checking, and diagnostic analysis.

---

## Objective
- Quantify the impact of trip distance on travel time  
- Measure how rush hour modifies average trip duration  
- Test whether distance behaves differently during rush hour using interaction terms  
- Diagnose why mean-based models are insufficient for time-of-day travel decisions  

---

## Dataset
- NYC Taxi Trip dataset  
- Target variable: `trip_duration` (seconds)  
- Key features:
  - `distance_km`
  - `is_rush`

The dataset contains no missing values and exhibits strong right-skew in raw trip duration.

---

## Methodology

### 1. Exploratory Data Analysis
- Visualized raw trip duration distribution
- Identified heavy right skew and long-tail behavior
- Applied log transformation to stabilize variance

### 2. Feature Engineering
- Log-transformed distance: `log1p(distance_km)`
- Binary rush hour indicator
- Interaction term:
  `log1p(distance_km) × is_rush`

### 3. Modeling
An Ordinary Least Squares (OLS) regression was implemented from scratch using the normal equation:

/ 
  log(T) = β0 + β1·log(1 + D) + β2·R + β3·(log(1 + D) × R)


This model estimates the conditional mean log trip duration.

---

## Evaluation and Diagnostics
- MAE and RMSE computed on log scale
- Residual analysis revealed:
  - Strong heteroscedasticity
  - Asymmetric error distribution
  - Structural limits on prediction accuracy

These diagnostics demonstrate why mean-based linear models cannot reliably support time-of-day travel decisions, despite fitting average trends well.

---

## Key Insights
- Distance is the dominant driver of trip duration
- Rush hour introduces an additional average delay
- Distance-related delays are amplified during rush hour (interaction effect)
- Variance and tail behavior dominate real-world travel decisions

---

## Limitations
- The model predicts average trip duration, not individual outcomes
- It does not model uncertainty, risk, or worst-case delays
- Time-of-day decisions require distribution-aware models (e.g., quantile regression)

---

## Future Work
- Introduce hour-of-day features
- Use quantile regression to model median and worst-case trip durations
- Compare mean-based vs risk-aware decision strategies

---

## One-Line Summary
An interpretable regression analysis that quantifies how distance and rush hour interact to affect average NYC taxi trip duration, and demonstrates why mean-based models are insufficient for time-of-day travel decisions.
