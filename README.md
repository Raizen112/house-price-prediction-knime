House Price Prediction — ML Regression Pipeline (KNIME)

End-to-end regression pipeline built in KNIME Analytics Platform that cleans a deliberately messy real estate dataset, engineers features, trains and compares two regression models, and exports evaluated predictions to Excel.

Overview

This project demonstrates a full data science workflow — not just modeling on a pre-cleaned dataset, but handling realistic, messy input data end to end:


Data ingestion of a raw, dirty CSV (invalid prices, impossible bedroom counts, missing values, duplicates)
Domain-rule-based cleaning, rather than blind statistical outlier removal
Feature engineering with one-hot encoding for categorical location data
Model training and comparison: Linear Regression vs. Random Forest
Evaluation using R², MAE, and RMSE
Export of predictions to a clean, readable Excel file


Built entirely as a visual workflow in KNIME — no code required to run it, just KNIME Analytics Platform.

Dataset

House_Price_Dirty.csv — ~4,630 rows of house sale records (Seattle/King County area) with intentionally realistic data quality issues:


Negative and placeholder prices (e.g. -$5,000, $99,999,999)
Impossible values (bedrooms = 99, negative bathrooms, construction years like 2999)
~8% missing values across several numeric columns
29 exact duplicate rows
No location/casing issues found on inspection (verified, not assumed)


Pipeline

The workflow is organized into 5 stages:

1. Data Ingestion
CSV Reader → initial Statistics view to profile missing values, ranges, and anomalies before any cleaning decisions are made.

2. Data Cleaning


Row Filter: removes rows with domain-invalid values (price ≤ 0, bedrooms outside 0–15, bathrooms < 0, sqft_living outside a sane range, yr_built outside 1800–2026)
Missing Value node: median imputation for numeric columns, most-frequent-value for strings — median chosen over mean to avoid distortion from remaining extreme-but-legitimate values
Duplicate Row Filter: removes exact duplicate records (all columns compared)


3. Feature Engineering


Column Filter: drops non-predictive columns (street, country, date)
One to Many: one-hot encodes city (44 categories)
Column Filter: drops one city dummy column to avoid the dummy variable trap for Linear Regression
Normalizer: min-max scales numeric features — explicitly excludes price, since normalizing the target breaks evaluation


4. Model Training


70/30 train/test split (random sampling, fixed seed for reproducibility)
Linear Regression Learner — trained on one-hot encoded features
H2O Random Forest Learner (Regression) — trained on the same features, using H2O's native categorical handling for city


5. Evaluation & Export


Numeric Scorer for each model: R², MAE, RMSE
Denormalizer applied before export so feature values in the output are real, readable numbers (not 0–1 scaled)
Excel Writer: predictions exported to Predictions.xlsx, one sheet per model


Results

MetricLinear RegressionRandom ForestR²0.6250.519MAE$147,096$152,381RMSE$273,155$261,831

Linear Regression slightly outperformed Random Forest on this dataset. A likely contributor is that the two models use different categorical encodings for city (one-hot for Linear Regression vs. H2O's native categorical handling for Random Forest) rather than an identical feature set.

Known Limitations


Linear Regression and Random Forest were trained/tested on independently sampled partitions rather than one shared split, so the comparison isn't a strictly controlled A/B test.
The two models don't use an identical feature representation for city, as noted above.


A stricter follow-up would standardize both the train/test split and the categorical encoding across models before comparing metrics.

Tech Stack


KNIME Analytics Platform
H2O (Random Forest node integration)
Excel Writer for output


Repo Contents


House_Price_Dirty.csv — raw input dataset
House_Price_Prediction_Pipeline.knwf — exported KNIME workflow
Predictions.xlsx — final output with predictions from both models
README.md — this file


How to Run


Install KNIME Analytics Platform
Install the H2O Machine Learning extension (KNIME Preferences → Install Extensions)
Import House_Price_Prediction_Pipeline.knwf (File → Import KNIME Workflow)
Point the CSV Reader node to your local copy of House_Price_Dirty.csv
Execute the full workflow

## 👤 Author

**Adam Benmoussa** — Engineering Student at ESITH Casablanca
Specialized in Business & Data Management | AI Automation & BI
