# Predicting Airline Delays with PySpark

**Which U.S. airports and airlines run late, and can we predict a high-delay month?**

Every flight delay has a cause: the airline, the weather, air traffic control, security, or an earlier flight arriving late. The U.S. government publishes this breakdown for every airport and airline each month. This project builds a PySpark pipeline that cleans that data, works out how delayed each airport and carrier is, labels the problem months, and trains a Random Forest to predict them.

> Built for a cloud and big data project at DCU. 

---

## What the pipeline does

1. **Loads** the raw CSV into a Spark DataFrame.
2. **Cleans it:** converts the numeric columns to the right type, fills missing airline and airport names with `UNKNOWN`, and drops rows with missing values.
3. **Builds new columns:**
   - `first_of_month`, a proper date made from the year and month
   - `delay_rate`, the share of arriving flights that were delayed by 15 minutes or more
   - `dominant_cause`, the delay cause with the biggest count
   - an anomaly flag for rows where delayed flights outnumber total flights
   - `HighDelay`, a 0/1 label that is 1 when the delay rate is above 5%
4. **Analyses it:** ranks airports and carriers by average delay rate.
5. **Saves it** as a Parquet file, a compressed columnar format that Spark reads quickly.
6. **Trains a model:** string indexing, feature assembly and scaling, then a Random Forest classifier (200 trees, depth 10) in a single Spark ML pipeline, with an 80/20 train/test split.
7. **Evaluates it** with AUC, accuracy, F1, a confusion matrix, an ROC curve and feature importances.

## The data

U.S. airline delay-cause data from the Bureau of Transportation Statistics (BTS). *(Add the link and the date range you downloaded.)* Each row is one airline at one airport for one month, so it is **monthly aggregated data, not individual flights**. After cleaning there were about **47,000 rows** (37,746 for training and 9,366 for testing), covering years up to at least 2025.

Key columns include flights arriving, flights delayed 15+ minutes, counts of delays by cause (carrier, weather, air traffic system, security, late aircraft), and the delay minutes for each cause.

---

## What it found

### Airports with the highest average delay rate

| Airport | Average delay rate |
|---|---|
| HGR | 30.8% |
| BQN | 30.7% |
| EAU | 30.6% |
| FMN | 30.2% |
| USA | 30.0% |

SFO also appears in the top ten at 29.1%.

### Airlines with the highest average delay rate

| Carrier | Average delay rate |
|---|---|
| AA | 27.6% |
| B6 | 27.3% |
| F9 | 25.4% |
| ZW | 23.3% |
| OH | 22.7% |

**A caution:** these are simple averages of each month's rate, not weighted by the number of flights, so a small airport with a few bad months can rank above a busy one.

---

## The model, and an honest look at its scores

The Random Forest on the test set scored:

| Metric | Value |
|---|---|
| AUC | 0.9993 |
| Accuracy | 0.9922 |
| F1 | 0.9920 |

**These numbers are too good to be believed, and the reason is data leakage.** The `HighDelay` label is calculated from `arr_del15 / arr_flights`, and both of those columns were also given to the model as inputs. The model is partly being handed the answer. The delay-cause counts (`carrier_ct`, `nas_ct` and so on) add up to `arr_del15`, so they leak it too. That is why `arr_del15` and `arr_flights` top the feature importance list (31.6% and 16.5%), followed by `carrier_delay` (14.2%) and `carrier_ct` (12.3%).

So the notebook's takeaway that operations matter more than weather is **not yet supported**. The ranking mostly shows how the label was built. A fair test would drop the leaked columns and predict a high-delay month from information available beforehand, such as the airline, the airport, the month and the number of flights.

Two further notes on reading the results:
- The 5% cut-off is low, since the average delay rate is around 20%, so most airport-months are labelled high delay. *(Check the class split and add it here.)*
- The precision (0.975) and recall (0.885) printed in the notebook are Spark's defaults, which report the **low-delay class (label 0)**, not the high-delay class.

## Known issues

- **Leakage**, as described above.
- **`dominant_cause` only works for carrier delays.** A bug in the loop that builds the cause label means only the first cause (carrier) is ever matched, so the column holds just "carrier" or "unknown". Fixing it means chaining every cause with `when`.
- **A 150-row sample (`flight_small`) is created but never used.**
- **One-hot encoders are defined but not used.** The model uses the string-indexed columns as numbers, which works for trees but treats airports as an ordered scale.

## What I would do next

- Remove the leaked columns and retrain to get honest scores
- Fix `dominant_cause`, and weight the airport and carrier averages by flights
- Predict next month's delay rate from earlier months, which is closer to a real use case
- Compare against Logistic Regression and gradient boosting
- Run the same pipeline on a cluster such as AWS EMR or Databricks to test it at scale

---

## Tech stack

Python · PySpark 3.5 (Spark SQL and Spark ML) · Pandas · Matplotlib · Seaborn · scikit-learn (ROC curve) · Google Colab · Parquet

## Run it yourself

The notebook was developed in **Google Colab** with Spark running locally (`local[*]`), not on a cluster. The same PySpark code could be run on EMR or Databricks, but that has not been tested here.

1. Open the notebook in Colab.
2. Upload `Airline_Delay_Cause.csv` to the Colab files.
3. Run the cells in order. The first cell installs `pyspark==3.5.1` and `findspark`.
4. The cleaned data is written to `flight_data.parquet`.

## Authors

Sumukha Sagar
School of Computing, Dublin City University

## License

MIT
