# NYC Taxi Trip Duration Analysis Report

## Executive Summary

This project analyzes NYC taxi trip duration data to understand the factors influencing how long taxi rides take. The analysis involves exploratory data analysis (EDA), feature engineering, and statistical correlations to identify key predictors of trip duration.

---

## 1. Dataset Overview

**Dataset Size:** 53,373 taxi trips
**Key Columns:**
- `trip_duration`: Duration in seconds (target variable)
- `vendor_id`: Taxi vendor identifier
- `pickup_datetime` & `dropoff_datetime`: Timestamps
- `passenger_count`: Number of passengers
- `pickup_longitude`, `pickup_latitude`: Pickup coordinates
- `dropoff_longitude`, `dropoff_latitude`: Dropoff coordinates
- `store_and_fwd_flag`: Flag indicating if trip was stored and forwarded

**Data Quality:** No null values detected in the initial dataset scan.

---

## 2. Target Variable Analysis

### 2.1 Raw Trip Duration Distribution

The raw trip duration shows a **heavily right-skewed distribution** with the following characteristics:

- **Distribution Pattern:** Single large concentration on the left, with a long tail extending to the right
- **Interpretation:** Most trips are relatively short, but a small number of extremely long trips stretch the distribution
- **Limitation:** This scale compresses most observations near zero, making it unsuitable for parametric statistical analysis

### 2.2 Log-Transformed Duration

To stabilize variance and enable linear modeling, a **log(1 + x) transformation** was applied:

**Results:**
- The log-transformed distribution becomes **approximately normal**
- Skewness is significantly reduced
- Extreme values are compressed while relative differences are preserved
- **This transformation is suitable for regression modeling and hypothesis testing**

---

## 3. Feature Engineering

### 3.1 Trip Distance Calculation (Haversine Formula)

**Methodology:**
Since trip distance was not directly provided, we calculated the great-circle distance between pickup and dropoff coordinates using the **Haversine formula**:

$$d = 2R \cdot \arcsin\left(\sqrt{\sin^2\left(\frac{\Delta\phi}{2}\right) + \cos(\phi_1)\cos(\phi_2)\sin^2\left(\frac{\Delta\lambda}{2}\right)}\right)$$

Where:
- R = Earth's radius (6,371 km)
- φ₁, λ₁ = Pickup latitude/longitude
- φ₂, λ₂ = Dropoff latitude/longitude

**Key Features:**
- **Accounts for Earth's curvature**, providing more accurate distances than Euclidean calculations
- **Distance measured in kilometers**, providing physically meaningful units
- Introduces a key explanatory variable strongly related to trip duration

### 3.2 Temporal Features

Extracted time-based features from pickup timestamps:

- **Hour:** Hour of day (0–23)
- **Weekday:** Day of week (Monday=0, Sunday=6)
- **Weekday Name:** Descriptive day names

These features enable analysis of hourly and daily patterns in congestion and trip duration.

### 3.3 Rush Hour Indicator

Defined binary rush-hour feature based on NYC peak traffic windows:

- **Morning Rush:** 7–10 AM
- **Evening Rush:** 4–7 PM

**Class Distribution:**
- Non-rush trips: 1,036,932 (71%)
- Rush-hour trips: 421,712 (29%)

---

## 4. Exploratory Data Analysis (EDA)

### 4.1 Spearman Correlation Analysis

Correlation matrix between key features and log-transformed trip duration:

| Feature | Correlation with log_duration |
|---------|-------------------------------|
| **distance_km** | **0.794** |
| passenger_count | 0.026 |
| hour | 0.031 |
| weekday | -0.030 |

**Key Insights:**

1. **Distance is the Dominant Predictor:** Trip distance shows a strong positive correlation (0.794) with trip duration. Longer trips consistently take more time.

2. **Weak Temporal Effects:** Hour and weekday show near-zero correlations, suggesting that time-of-day and day-of-week have minimal direct effect on trip duration when distance is considered.

3. **Passenger Count Effect:** Almost no correlation (0.026), indicating passenger count alone does not meaningfully predict trip duration.

### 4.2 Rush Hour vs Non-Rush Trip Duration

**Summary Statistics (Log-Transformed Duration):**

| Metric | Non-Rush | Rush |
|--------|----------|------|
| Mean | 6.459 | 6.486 |
| Median | 6.494 | 6.506 |
| Std Dev | 0.791 | 0.807 |

**Observations:**

- **Central Tendencies are Similar:** Rush hour trips have only slightly higher mean (0.027 higher) and median durations on the log scale
- **Higher Variability in Rush Hours:** Standard deviation is slightly higher during rush hours (0.807 vs 0.791), suggesting more extreme values
- **Heavier Right Tail During Rush Hours:** While log-transformed distributions appear similar, the raw scale analysis reveals a heavier right tail during rush hours, indicating increased likelihood of longer trip durations during peak traffic

---

## 5. Key Findings

1. **Distance Dominates Duration:** Trip distance is by far the strongest predictor of trip duration (r = 0.794), indicating that spatial separation is the primary driver of time.

2. **Limited Rush Hour Effect:** Traditional rush hours (7–10 AM, 4–7 PM) show minimal impact on average trip duration when distance is controlled for, though there is increased variability in outcomes.

3. **Temporal Patterns Are Subtle:** Hour and weekday show negligible direct correlations with trip duration, suggesting that congestion effects are secondary to the fundamental distance constraint.

4. **Data Quality:** The dataset contains no missing values and covers a diverse range of trip durations and distances, providing a solid foundation for further modeling.

---

## 6. Improvements & Recommendations

### 6.1 Data Enhancement

1. **External Traffic Data Integration**
   - Incorporate real-time traffic speed data from Google Maps API or similar sources
   - Add road network characteristics (highway vs. street, number of intersections)
   - Include weather conditions (precipitation, visibility) that may impact travel times
   - **Impact:** These would likely provide stronger predictive signals than hour/weekday alone

2. **NYC Infrastructure Features**
   - Add borough/zone information for origin and destination
   - Include proximity to major traffic chokepoints (bridges, tunnels)
   - Incorporate special events or construction zones
   - **Impact:** Could explain remaining variance and improve model accuracy

3. **Enhanced Temporal Granularity**
   - Expand beyond binary rush/non-rush to hourly or 15-minute bins
   - Create interaction features between time and distance
   - Consider seasonal patterns (holidays, summer vs. winter)
   - **Impact:** Might reveal non-linear time-of-day effects

### 6.2 Feature Engineering Improvements

1. **Distance-Based Segmentation**
   - Create categorical distance bins (short: <1 km, medium: 1-5 km, long: >5 km)
   - Fit separate models for each distance category
   - **Rationale:** Trip duration dynamics may differ significantly by trip length

2. **Interaction Features**
   - Distance × Rush Hour: Interaction term to capture how congestion scales with trip length
   - Distance × Vendor: Different vendors may have different patterns
   - **Impact:** Could reveal non-additive effects currently masked

3. **Speed-Based Features**
   - Calculate average speed (distance / duration)
   - Identify unusually slow/fast trips by distance category
   - **Insight:** Flag anomalies and understand trip efficiency variations

### 6.3 Advanced Modeling Approaches

1. **Multi-Model Strategy**
   - Build separate models for short vs. long trips
   - Use ensemble methods combining distance-based and temporal models
   - Implement quantile regression to predict trip duration percentiles
   - **Benefit:** Account for heterogeneous relationships across trip types

2. **Spatial Analysis**
   - Perform geospatial clustering to identify congestion zones
   - Build zone-specific duration prediction models
   - **Benefit:** Capture location-specific congestion patterns

3. **Time-Series Anomaly Detection**
   - Use time-series decomposition to separate trend, seasonality, and anomalies
   - Identify unusual traffic patterns and events
   - **Benefit:** Better understanding of exceptional trips

### 6.4 Data Quality & Outlier Handling

1. **Outlier Investigation**
   - Current analysis shows some trips with minimal coordinates (all NaN)
   - Investigate trips with extreme durations relative to distance
   - Consider whether outliers represent data errors or genuine phenomena
   - **Action:** Implement robust outlier handling strategies based on findings

2. **Missing Value Imputation**
   - Monitor for missing data in temporal features
   - Develop imputation strategies for vendor or dropoff info if needed
   - **Preventive:** Maintain data quality logs

### 6.5 Business Intelligence Enhancements

1. **Predictive Deployment**
   - Build production-ready model to estimate trip duration for users booking rides
   - Implement confidence intervals around predictions
   - **Use Case:** Better customer expectations and surge pricing

2. **Driver Intelligence**
   - Identify route efficiency opportunities
   - Recommend optimal pickup/dropoff zones
   - **Use Case:** Operational optimization

3. **Demand Forecasting**
   - Extend analysis to predict trip frequency patterns
   - Combine duration insights with volume predictions
   - **Use Case:** Resource allocation and driver scheduling

---

## 7. Methodology Notes

- **Correlation Method:** Spearman rank correlation (non-parametric) used to capture monotonic relationships without assuming linearity
- **Transformation Rationale:** Log transformation applied to stabilize variance and enable parametric statistical inference
- **Geographic Calculation:** Haversine formula preferred over Euclidean distance for its geographic accuracy

---

## 8. Conclusion

The analysis reveals that **trip distance is the dominant predictor of taxi trip duration**, with a correlation of 0.794. Traditional congestion proxies like rush-hour time windows show minimal direct correlation when distance is considered, though they exhibit increased variability. To improve predictive accuracy, future work should focus on incorporating external traffic data, weather information, and enhanced spatial features. The current dataset provides a solid foundation but would significantly benefit from integration with real-time traffic and NYC infrastructure data.

---

## Repository Structure

```
.
├── README.md                          # This file
├── notebooks/
│   └── NYC_DA.ipynb                   # Main analysis notebook
└── data/
    └── NYC.csv                         # Dataset (not included)
```

## Requirements

- Python 3.7+
- pandas
- numpy
- matplotlib
- scipy (for Spearman correlation)

## Usage

Open `notebooks/NYC_DA.ipynb` in Jupyter Notebook to explore the analysis step-by-step.

---

*Last Updated: July 2026*
