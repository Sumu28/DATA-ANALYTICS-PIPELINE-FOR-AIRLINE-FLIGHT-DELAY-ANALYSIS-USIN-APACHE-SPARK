## Overview

This project implements a scalable, cloud-ready data analytics pipeline to analyze airline flight delays using large-scale U.S. aviation data. Apache Spark and PySpark are used to perform distributed data processing, feature engineering, and machine learning to identify high-delay periods efficiently.

## Problem Statement

Flight delays significantly impact airline operations, customer satisfaction, and overall network efficiency.
The goal of this project is to identify key factors contributing to flight delays and to predict high-delay months using a big-data processing framework.

## Dataset

Source: U.S. Bureau of Transportation Statistics (BTS)

Dataset: On-Time Performance – Delay Causes

Link: https://www.transtats.bts.gov/OT_Delay/OT_DelayCause1.asp

Description:
Monthly aggregated U.S. domestic flight arrival data including total flights, delayed flights, and breakdowns by delay causes such as weather, carrier, NAS, security, and late aircraft.

The raw dataset is not included in this repository due to size and licensing constraints.

## Data Processing Pipeline

All data processing is performed using Apache Spark (PySpark).

Pipeline stages:

1. Data ingestion from CSV into Spark DataFrames

2. Data cleaning and validation

3. Feature engineering (delay rate, dominant delay cause, binary delay label)

4. Categorical encoding using StringIndexer and OneHotEncoder

5. Feature vector assembly

6. Train–test split

7. Machine learning using Random Forest Classifier

## Machine Learning

Model: Random Forest Classifier

Target Variable: HighDelay (binary indicator of high delay months)

Evaluation: Confusion Matrix, ROC Curve

Key Insights:
Late aircraft and NAS delays are the most influential contributors to high-delay periods.

## Tools & Technologies

Apache Spark

PySpark

Spark MLlib

Python

Jupyter Notebook

Cloud-ready architecture (AWS EMR / Databricks compatible)

## Outputs

Cleaned and transformed datasets

Feature-engineered Parquet files

Trained Random Forest model

Feature importance analysis

ROC and confusion matrix visualizations

## License & Data Usage

The code in this repository is licensed under the MIT License.

The dataset is provided by the U.S. Bureau of Transportation Statistics (BTS) and is subject to their data usage terms.

Raw data is not redistributed in this repository.
