
---

# **Flight Delay Prediction — Distributed Machine Learning Pipeline (PySpark + AWS)**

---

##  **Introduction**

Flight delays significantly impact airline operations, airport efficiency, and passenger experience.
This project demonstrates a **scalable, distributed machine learning workflow** using **PySpark** and **AWS** to predict **flight arrival delays**.

Core goals:

* Process **millions of flight records** using PySpark
* Build and evaluate a flight delay prediction model using **MLlib**
* Explore how **S3** + **Athena** streamline ETL and querying
* Show trade-offs in operational modeling for real-world airline workflows

All work is performed inside **a single Jupyter Notebook** that includes the complete pipeline:

**EDA → Preprocessing → Modeling → Evaluation**

---

##  **Dataset Overview**

The dataset consists of U.S. domestic airline performance records, including:

* Flight timestamps
* Carrier identifiers
* Origin–destination routes
* Delay reasons

**Dataset Size:** Millions of rows (Jan 2020 – Feb 2025)

### **Data Source:**

Publicly available U.S. airline delay datasets
(Stored originally on AWS S3 → Parquet → Athena)

---

## **Feature Categories**

### **✔ Pre-Flight Features (Valid Predictors)**

These are known **before takeoff**:

| Column    | Description             |
| --------- | ----------------------- |
| `month`   | Month of flight (1–12)  |
| `carrier` | Airline carrier code    |
| `route`   | Origin–Destination pair |

---

### ** Post-Flight Features (Leakage – Only for EDA)**

These are known **after landing**, therefore **cannot** be used for real inference:

| Column                | Description                 |
| --------------------- | --------------------------- |
| `arr_delay`           | Arrival delay (minutes)     |
| `carrier_delay`       | Carrier-related delay       |
| `weather_delay`       | Weather-related delay       |
| `nas_delay`           | NAS delay                   |
| `security_delay`      | Security-related delay      |
| `late_aircraft_delay` | Delay from inbound aircraft |

> **Transparency Note:**
> Early experiments used these leakage fields to explore correlations & feature importance.
> These versions are included in the notebook for instructional purposes but **not valid for production**.

---

##  **Data Preprocessing (PySpark)**

Performed fully in PySpark:

* Convert delay columns from string → numeric
* Fill missing values with safe defaults
* Encode categorical features using **StringIndexer**
* Assemble final feature vector via **VectorAssembler**

---

##  **Exploratory Data Analysis (EDA)**

Key findings:

* Major delay contributors: **late aircraft**, **NAS**, **carrier**
* Strong class imbalance between delayed vs on-time flights
* `arr_delay` is highly correlated with “delayed” status (leakage)
* Delay patterns vary significantly by carrier and route

---

##  **Machine Learning Model**

Model: **RandomForestClassifier (PySpark MLlib)**

**Why Random Forest?**

* Handles large distributed datasets
* Captures non-linear relationships
* Provides interpretable feature importance
* Scales efficiently on EMR clusters

### **Pipeline Steps**

1. Encode categorical fields
2. Assemble vector features
3. Fit RandomForest model
4. Evaluate model performance

---

##  **Model Performance (Experimental)**

### **Confusion Matrix**

|                        | Predicted On-Time (0) | Predicted Delayed (1) |
| ---------------------- | --------------------- | --------------------- |
| **Actual On-Time (0)** | 10,051                | 8,330                 |
| **Actual Delayed (1)** | 2,455                 | 28,258                |

### **Interpretation**

* **High recall** for delayed flights (good for airline planning)
* False positives exist, but overpredicting delays is better than missing them
* Model is *not* production-ready due to leakage variables
  (included intentionally for educational analysis)

---

##  **Feature Importance (Experimental)**

*(Includes leakage)*

| Feature               | Importance (%) |
| --------------------- | -------------- |
| `arr_delay`           | 39.0           |
| `nas_delay`           | 19.1           |
| `late_aircraft_delay` | 16.1           |
| `carrier_delay`       | 15.0           |
| `route_indexed`       | 4.8            |

---

##  **Cloud Architecture**

### **Data Flow**

```
Raw CSV → S3 → PySpark (EMR) → Parquet → Athena → ML Model
```

### **Future Scalable Architecture**

```
Airflow → EMR Jobs → Model Training → Batch or Real-Time Scoring
```

---

##  **Challenges & Solutions**

| Challenge                   | Solution                                       |
| --------------------------- | ---------------------------------------------- |
| Class imbalance             | Use Recall / F1; consider SMOTE/undersampling  |
| Leakage variables           | Identified early; removed for production       |
| High-cardinality categories | Used `handleInvalid="keep"` + larger `maxBins` |
| Very large dataset          | Distributed PySpark processing                 |

---

##  **Conclusion**

This project demonstrates how **PySpark + AWS** enable scalable ETL and machine learning for large flight delay datasets.

While the **experimental** model includes leakage for educational clarity, the notebook clearly outlines a roadmap to a **production-ready pre-flight prediction system**.

---

##  **Future Work**

Planned improvements:

* Remove all leakage variables from modeling
* Address class imbalance (SMOTE, undersampling)
* Hyperparameter tuning with CrossValidator
* Add XGBoost & LightGBM models
* Build real-time inference with AWS Lambda
* Add external data sources (weather API)
* Add Airflow orchestration for automated pipelines

---


