# Project Notes

## Day 1

### Problem

Supply chain businesses need to predict future demand
and use those predictions to make better inventory and
replenishment decisions.

### Important Concepts

1. Demand Forecasting

Demand forecasting estimates future customer demand
using historical and relevant explanatory data.

2. Prediction vs Decision

The ML model predicts future demand.
The decision engine uses predictions to recommend actions.

3. Production Thinking

A real ML system consists of data, models, APIs,
infrastructure, monitoring and business logic rather
than only a model notebook.

### Dataset observation

The M5 sales data is stored in wide format, with one row representing an item-store series and daily sales represented by d_1, d_2, etc.

### Important finding

In the 1,000-row sample, 67.21% of sales cells were zero. This indicates substantial sparse or intermittent demand and means forecasting models and evaluation metrics must be chosen carefully.

### Important distinction

The zero percentage calculated on a sample is not necessarily the zero percentage of the entire dataset or total retail demand.

## Day 2 — Data Transformation

### Objective

Convert the M5 sales data from wide format to long format
and map the daily sales columns to actual calendar dates.

### Forecasting Grain

One row represents the sales of one product in one store
on one date.

Key:
item_id + store_id + date

### Wide Format

The M5 sales dataset stores daily sales in columns such as:

d_1, d_2, d_3, ...

### Long Format

We transform the data into:

date | item_id | store_id | sales

### Why Long Format?

Long format is easier for:
- time-series analysis
- SQL aggregation
- feature engineering
- machine learning
- visualization

### Important Concept — Grain

Grain defines what one row in a dataset represents.

### Important Concept — Data Transformation

Raw data is transformed into an analytical structure
that can be consumed by downstream ML and business logic.

### Planned Pipeline

M5 Sales
→ Melt / Unpivot
→ Long Format
→ Calendar Mapping
→ Price Mapping
→ Clean Analytical Dataset


### Composite Key Validation

Price data is joined to sales data using:

item_id + store_id + wm_yr_wk

The price table contains no duplicate records for this
composite key.

After merging, the row count remained unchanged:

Before: 1,913,000
After:  1,913,000

This confirms that the merge did not multiply sales observations
and preserved the intended dataset grain.

### Price Data Investigation — Product-Level Check

A product-level inspection was performed for HOBBIES_1_001
at store CA_1.

The inspection showed that:
- Some dates have missing sell_price values.
- Later dates have valid prices.
- Sales can be zero even when a valid price exists.
- The product price changes over time (for example, 8.26 to 8.38).

This confirms that:
- sales = 0 does not mean price = 0.
- missing sell_price should not automatically be replaced with 0.
- missing price and zero sales represent different concepts.

The observed missing-price statistics are based on the
1000-row sales sample used during transformation, not the
complete M5 dataset.


### Sales Distribution

For the 1000-row sales sample, the transformed dataset
contains 1,913,000 product-store-day observations.

Sales statistics:
- Mean: 0.92
- Median: 0
- 75th percentile: 1
- Maximum: 294

The median sales value is 0, indicating that a large proportion
of product-store-day observations have zero demand.

The maximum value of 294 compared with the median of 0 indicates
a highly right-skewed sales distribution.

This suggests that the dataset contains sparse/intermittent
demand patterns, which will be important when selecting
forecasting features, models, and evaluation metrics.

Zero-sales observations account for approximately 67.21%
of the product-store-day observations in the 1000-row sample.

This indicates that demand is sparse/intermittent in a large
portion of the observations.

The percentage refers specifically to zero-valued sales
observations in the transformed sample and should not be
interpreted as the percentage of products with no sales.


### Zero-Sales Analysis

Total product-store-day observations: 1,913,000

Zero-sales observations: 1,285,760 (67.21%)
Non-zero sales observations: 627,240 (32.79%)

Therefore, approximately two-thirds of the observations
in the 1000-row sample have zero recorded sales.

This indicates a sparse/intermittent demand pattern.

### Price Distribution

Among observations with an available sell_price:

- Count: 1,520,496
- Mean price: 5.22
- Median price: 3.97
- Standard deviation: 4.33
- Minimum price: 0.01
- Maximum price: 30.98

The mean is higher than the median, indicating that higher-priced
observations increase the average price.

The minimum and maximum values are not automatically treated as
errors or removed as outliers. Their validity should be evaluated
using business and product context before any outlier treatment.

### Feature Engineering

Feature engineering is the process of creating useful input
variables from existing raw or processed data so that a machine
learning model can learn relevant patterns.

For demand forecasting, the date column contains useful temporal
information. Therefore, calendar features such as year, month,
day of week, and weekend indicator are extracted from date.

Features that can influence demand but are not derived from the target's previous values.

## Sample representativeness
A sample is useful for development, but its limitations must be understood before generalizing conclusions to the full dataset.

Our current sample represents:
CA
└── 1,000 product/store rows
    └── 1,913 days
it doesnot represent CA+TX+WI together

## Defining our forecasting model
One assumption for this project model : today's price must be known or planned before the forecast is made.

### Important concept — Target leakage
The target is the value we want to predict. Here, it's sales.
We must never include the actual same-day sales column among the model's input features. Otherwise, the model would be given the answer it's supposed to predict.

### Time-based-splitting :
We must not randomly shuffle our time-series observations. Instead, we'll train on earlier dates and evaluate on later dates to simulate forecasting future demand.
Remove observations without sufficient sales history Our lag and rolling features require historical observations. The first 28 days of each product-store series do not have a complete 28-day history. For this initial baseline, we'll exclude those rows. We will not drop rows merely because their price or event fields are missing.

## What does chronologically mean?
Chronologically means arranging data in time order, from oldest to newest.
For our project, the dates start in January 2011 and continue until April 2016.
For example:
Date	Sales	Purpose
Jan 29, 2011	0	Training
Jan 30, 2011	2	Training
Jan 31, 2011	1	Training
...	...	...
Early April 2015	3	Around the split
...	...	...
April 24, 2016	4	Validation

Why do we split this way?
Imagine you're building a model today to predict tomorrow's demand.
You can use past data to learn patterns, but you cannot use tomorrow's actual sales to train the model before predicting them.
So our approach is:
Training data — Earlier dates
Approximately 80% of the dates. The model learns from historical demand patterns.

Train the model

Validation data — Later dates
Approximately 20% of the dates. We test how well the trained model predicts demand in a later period.

## Visualization Observation

The 28-day rolling average shows an overall upward trend in the selected sample.



Daily sales have substantial short-term variability.



A possible weekday pattern needs further investigation.



Sharp drops to zero around year boundaries require data-quality validation.

## Day 4 — Exploratory Data Visualization

### Objective
Understand demand trends, weekday patterns, and monthly sales variation using the analytical sample.

### 1. Daily Sales Trend
- Aggregated sales across the 1,000 sampled product series by date.
- Visualized daily total sales and a 28-day rolling average.
- The rolling average shows an overall upward trend during the observed period.
- Daily sales exhibit substantial short-term fluctuations.
- Several dates show zero total sales and require further data-quality investigation.

### 2. Average Daily Sales by Weekday
- Aggregated sales by date before calculating weekday averages.
- Saturday and Sunday show the highest average combined daily sales.
- Average sales are lower during the middle of the week.
- Friday shows a rise relative to the middle of the week.
- These patterns apply to the current California-only sample, not necessarily to every product.

### 3. Average Daily Sales by Calendar Month
- Calculated average daily total sales for each calendar month across the available years.
- August appears to have the highest average daily sales.
- May and December are among the lower months in the visualization.
- This analysis does not isolate monthly seasonality because the overall sales trend changes over time.
- Further analysis is required before treating these differences as stable seasonal patterns.

### Important Considerations
- The analysis uses 1,000 sampled product series from California.
- Aggregate sales can hide differences between individual products.
- Visualization findings should be validated before turning them into modeling assumptions.
- Zero-total dates require investigation.
- Monthly seasonality should be examined while accounting for long-term trends.

### Next Steps
- Investigate dates with zero total sales.
- Analyze monthly patterns by year.
- Continue exploratory analysis of demand distributions and event effects.
- Compare forecasting models against the established baseline metrics.