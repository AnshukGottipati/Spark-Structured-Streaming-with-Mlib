# Handson-L10-Spark-Streaming-MachineLearning-MLlib

# **Real-Time Taxi Fare Prediction – Task 4 & Task 5**

This repository contains the solutions for:

* **Task 4: Basic Streaming Ingestion & Real-Time Fare Prediction**
* **Task 5: Anomaly Detection using Prediction Deviation**

Both tasks use **Apache Spark Structured Streaming** and **Spark MLlib** to train a regression model offline and apply it to live streaming data from a socket source.

---

#  **Repository Structure**

```
.
├── task4_fare_prediction.py       # Offline model training + real-time prediction
├── task5_anomaly_detection.py     # Includes deviation scoring for anomalies
├── training-dataset.csv           # Provided dataset
├── models/
│   └── fare_model/                # Saved Linear Regression model
├── README.md
└── screenshots/
    ├── task4_output.png
    ├── task5_output.png
```

> NOTE: The `screenshots/` folder can be renamed or reorganized as needed.
> Insert your own output images where indicated below.

---

#  **Task 4: Basic Streaming Ingestion & Parsing**

###  Goal

Train a **Linear Regression** model using historical ride data and apply it to **incoming streaming trips** to predict taxi fares in real time.

### ✔Steps Implemented

1. **Offline Model Training**

   * Reads `training-dataset.csv`
   * Casts `distance_km` and `fare_amount` columns to `DoubleType`
   * Uses a `VectorAssembler` to create `features`
   * Trains a **Linear Regression** model to predict `fare_amount`
   * Saves model → `models/fare_model/`

2. **Streaming Inference**

   * Reads live JSON input from socket: `nc -lk 9999`
   * Parses JSON using a defined schema
   * Uses same `VectorAssembler` to prepare streaming features
   * Loads the saved regression model
   * Predicts `predicted_fare` for each incoming ride

###  **Expected Output (Task 4)**

Columns printed in streaming output:

| Column           | Description                                  |
| ---------------- | -------------------------------------------- |
| `trip_id`        | Unique trip identifier                       |
| `driver_id`      | Driver number                                |
| `distance_km`    | Trip distance                                |
| `fare_amount`    | Actual fare                                  |
| `predicted_fare` | Model’s predicted fare                       |
| `deviation`      | Distance between actual & predicted (Task 5) |

---

##  **Task 4 – Output Screenshot**

Task 4:
-------------------------------------------
Batch: 6
-------------------------------------------
+------------------------------------+---------+-----------+-----------+-----------------+-----------------+
|trip_id                             |driver_id|distance_km|fare_amount|predicted_fare   |deviation        |
+------------------------------------+---------+-----------+-----------+-----------------+-----------------+
|a9f13e2a-c7c3-4e5b-b18d-ac97d5e4dab5|31       |29.19      |83.61      |90.97414083595437|7.364140835954373|
+------------------------------------+---------+-----------+-----------+-----------------+-----------------+



---

#  **Task 5: Deviation-Based Anomaly Detection**

###  Goal

Extend Task 4 by computing the **absolute deviation** between actual fare and predicted fare:

```
deviation = ABS(fare_amount - predicted_fare)
```

High deviation indicates a possible anomaly such as:

* Overcharging
* Incorrect sensor distance
* Fraudulent fare manipulation

###  Enhancements from Task 4

* Adds a column `deviation` to the streaming output
* Still operates in real-time using the same trained model
* Outputs structured, append-mode logs with anomaly indicators

### **Expected Output (Task 5)**

Example row:

```
+------------------+----------+------------+-----------+--------------+----------+
| trip_id          | driver_id| distance_km| fare_amount | predicted_fare | deviation |
+------------------+----------+------------+-----------+--------------+----------+
```

---

## **Task 5 – Output Screenshot**


Task 5:
-------------------------------------------                                     
Batch: 85
-------------------------------------------
+-------------------+-------------------+-----------------+-----------------------+
|window_start       |window_end         |avg_fare         |predicted_next_avg_fare|
+-------------------+-------------------+-----------------+-----------------------+
|2025-11-26 19:42:00|2025-11-26 19:47:00|76.24856666666669|92.39431034482759      |
+-------------------+-------------------+-----------------+-----------------------+

---

# **How to Run**

###  Start the streaming server

In a separate terminal:

```bash
python3 data_generator.py
```

Then send valid JSON per line:

```json
{"trip_id":"t1", "driver_id":12, "distance_km":5.1, "fare_amount":18.0, "timestamp":"2025-01-10 12:30:00"}
```

---

###  Run Task 4 script

```bash
python task4.py
```

###  Run Task 5 script

```bash
python task5.py
```

---


# Final Explanations/Approach
