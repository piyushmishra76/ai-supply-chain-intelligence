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