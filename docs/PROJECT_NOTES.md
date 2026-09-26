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