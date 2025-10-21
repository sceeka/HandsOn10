# Hands-on L10: Spark Structured Streaming + Machine Learning with MLlib

## Overview

This project implements a **real-time ride-sharing analytics pipeline** using **Apache Spark Structured Streaming** and **Spark MLlib**.
It is divided into two main components:

* **Task 4 – Real-Time Fare Prediction:**
  Builds a regression model to predict ride fares based on trip distance.
* **Task 5 – Time-Based Fare Trend Prediction:**
  Applies a time-series regression model to forecast average fare trends within 5-minute time windows.

Both modules utilize live ride events streamed via a Python socket server.

---

## Repository Structure

```
Handson-L10/
├── data_generator.py
├── task4.py
├── task5.py
├── training-dataset.csv
├── models/
│   ├── fare_model/
│   └── fare_trend_model_v2/
├── outputs_task4.txt
├── outputs_task5.txt
└── README.md
```

---

## Setup Instructions

### Install Dependencies

```bash
pip install pyspark faker
```

### Start the Data Stream

Run the data generator in **Terminal 1**:

```bash
python3 data_generator.py
```

Expected output:

```bash
Streaming data to localhost:9999...
Sent: {"trip_id":"...","driver_id":17,"distance_km":23.45,"fare_amount":67.20,"timestamp":"2025-10-21 17:30:00"}
```

The script streams continuous JSON events to the Spark listener at `localhost:9999`.

---

## Task 4 – Real-Time Fare Prediction

### Objective

Train and deploy a **Linear Regression model** to predict fare amounts based on `distance_km` in real time.

### Model Training

* Loads **training-dataset.csv**
* Converts columns to numerical types
* Assembles features using:

  ```python
  VectorAssembler(inputCols=["distance_km"])
  ```
* Trains a **LinearRegression** model
* Saves the trained model in `models/fare_model`

### Streaming Inference

* Reads live JSON records from the socket stream
* Assembles features dynamically
* Applies the trained model for fare prediction
* Adds a new column:

  ```
  deviation = |actual_fare − predicted_fare|
  ```
* Outputs continuous results to console and file.

### Run Command

Execute the script in **Terminal 2** (keep data generator active):

```bash
python3 task4.py | tee outputs_task4.txt
```

### Sample Output

```bash
trip_id | driver_id | distance_km | fare_amount | predicted_fare | deviation
---------------------------------------------------------------------------
cf30c1b1-d090-42bd-82e5-6ff59a12bab5 | 72 | 39.93 | 134.18 | 88.33 | 45.85
ef861f11-0729-41fe-9bb3-400f890ae604 | 53 | 21.96 | 107.17 | 92.75 | 14.41
28758d09-c137-47a5-8a3a-cf948ccdb511 | 93 | 18.03 | 81.19  | 93.71 | 12.52
```

---

## Task 5 – Time-Based Fare Trend Prediction

### Objective

Predict upcoming **average fare trends** using 5-minute window aggregations and temporal features such as `hour_of_day` and `minute_of_hour`.

### Model Training

* Aggregates historical ride data into **5-minute windows**
* Computes the average fare (`avg_fare`) per window
* Adds time-based features
* Trains a regression model to forecast the **next window’s average fare**
* Saves the model as `models/fare_trend_model_v2`

### Streaming Inference

* Reads live streaming data from the socket
* Applies **sliding window aggregation** (5 minutes, sliding every 1 minute)
* Predicts the next average fare trend for each window in real time.

### Run Command

Execute the script in **Terminal 3**:

```bash
python3 task5.py | tee outputs_task5.txt
```

### Sample Output

```bash
window_start          | window_end            | avg_fare | predicted_next_avg_fare
-------------------------------------------------------------------------------
2025-10-21 21:29:00   | 2025-10-21 21:34:00   | 76.05    | 92.39
2025-10-21 21:30:00   | 2025-10-21 21:35:00   | 79.12    | 90.81
```

---

## Key Concepts and Components

| Concept                        | Description                                                                 |
| ------------------------------ | --------------------------------------------------------------------------- |
| **Spark Structured Streaming** | Enables real-time ingestion and processing of continuous ride data streams. |
| **VectorAssembler**            | Combines numeric columns into feature vectors for MLlib models.             |
| **LinearRegression (MLlib)**   | Performs regression analysis for fare and trend predictions.                |
| **Window Aggregations**        | Groups streaming data into fixed-size time windows for trend analysis.      |
| **Pipeline Consistency**       | Maintains identical preprocessing steps during both training and inference. |

---

## Screenshots

📸 **Task 4 – Real-Time Fare Prediction Output**
📸 **Task 5 – Time-Based Fare Trend Output**

---

## Submission Checklist

✅ `data_generator.py` streams ride data in JSON format
✅ `task4.py` trains and performs real-time fare prediction
✅ `task5.py` trains and predicts fare trends over time windows
✅ Models (`fare_model`, `fare_trend_model_v2`) saved successfully
✅ Output logs (`outputs_task4.txt`, `outputs_task5.txt`) included
✅ Screenshots and final README attached

---

## Results

* **Task 4:** Successfully generated continuous fare predictions with deviation values.
* **Task 5:** Accurately captured and forecasted average fare trends across real-time time windows.

Together, these tasks demonstrate a **complete, end-to-end real-time machine learning pipeline** using **Spark Structured Streaming and MLlib**.
