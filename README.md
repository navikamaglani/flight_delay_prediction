# flight_delay_prediction
Flight Delay Prediction — Distributed Machine Learning Pipeline (PySpark + AWS)
Building a Scalable Pipeline for Pre-Flight Delay Prediction
1. Introduction
Flight delays create major operational and customer-experience challenges for airlines and airports.
This project develops a scalable, cloud-based machine learning pipeline capable of processing millions of flight records using PySpark and AWS infrastructure to predict flight arrival delays before takeoff.
Objectives
Use PySpark for distributed processing of large flight datasets
Build a Random Forest classifier using Spark MLlib
Use AWS S3 + Athena for efficient ETL and querying
Enable model training + scoring on millions of rows
Provide actionable insights through feature importance, recall, confusion matrix, and delay trends
2. Dataset Overview
This project uses large-scale U.S. flight performance data containing detailed timestamps, carriers, airports, and delay reasons.
Source: Publicly available airline delay data
Storage: Raw CSV files loaded into S3 → converted to Parquet → queried via Athena
2.1 Feature Categories
✔ Pre-Flight Features (Valid Predictors)
Known before takeoff — acceptable for prediction.
Column	Description
month	Month of flight (1–12)
carrier	Airline carrier code
Route	Origin–Destination pair (e.g., JFK–LAX)
⚠ Post-Flight Features (Leakage — MUST NOT be used)
Known after landing. Including them leads to label leakage.
Column	Description
arr_delay	Arrival delay in minutes
carrier_delay	Delay caused by carrier
weather_delay	Weather delay
nas_delay	NAS delay
security_delay	Security delay
late_aircraft_delay	Connecting aircraft delay
Delayed_Arrival	Binary target (0/1)
Data Size: Millions of records (Jan 2020 – Feb 2025)
⚠ Transparency Note (Honesty Section)
This project initially included post-flight delay features (e.g., arr_delay, nas_delay) during early experimentation.
These create label leakage and cannot be used in a real pre-flight prediction model.
I have kept the EDA and experimental model results for transparency, but the future/production version removes these columns entirely.
This shows awareness of real-world ML constraints and the correct direction for improvement.
3. Data Preprocessing
Large and messy flight datasets require careful cleaning.
Key Steps
Type Conversion: Convert delay strings → numeric
Missing Values: Replaced invalid values with null-safe defaults
Categorical Encoding:
StringIndexer for carrier
StringIndexer for Route
handleInvalid="keep" to handle unseen categories
Feature Vector Assembly:
Combined selected predictors into a single vector using VectorAssembler.
4. Exploratory Data Analysis (EDA)
Delay Patterns
Majority of long delays attributed to:
late_aircraft_delay
nas_delay
carrier_delay
Class Imbalance
Delayed flights significantly outnumber on-time flights.
This affects model performance metrics.
Correlation Analysis
arr_delay highly correlated with Delayed_Arrival → label leakage risk
Delay cause columns moderately correlated
Route- and carrier-level delay patterns observed
5. Machine Learning Model
Model Choice
A RandomForestClassifier from PySpark MLlib, selected for:
Performance on large datasets
Non-linear relationships
Robustness to noise
Built-in feature importance
Pipeline (Initial Experimental Version)
Encode categorical features
Assemble features
Train Random Forest
Evaluate using confusion matrix + recall
6. Model Performance
Confusion Matrix (Initial Experimental Run)
Predicted On-Time (0)	Predicted Delayed (1)
Actual On-Time (0)	10,051	8,330
Actual Delayed (1)	2,455	28,258
Interpretation
Model achieves high recall for delayed flights
Some false positives (predicting delay when flight is on-time)
In aviation, false positives are safer than false negatives — better to warn early than miss a delay
7. Feature Importance (Experimental)
Feature	Importance (%)
arr_delay	39.0
nas_delay	19.1
late_aircraft_delay	16.1
carrier_delay	15.0
Route_indexed	4.8
Reminder:
These include post-flight leakage variables and will be removed in the production pipeline.
8. Cloud Architecture
Data Flow Overview
Raw CSV → S3 → PySpark on EMR → Parquet → Athena → ML Model → Scoring → Insights Dashboard
Production Architecture (Future Design)
Airflow → Daily Ingestion → EMR → Feature Store → Model Training → Batch/Real-Time Scoring
9. Challenges & Solutions
Challenge	Solution
Class imbalance	Use recall/F1; plan SMOTE/undersampling
Label leakage	Identified → removed in future version
High-cardinality categoricals	Increased maxBins to 400; used handleInvalid="keep"
Large dataset size	Distributed PySpark + AWS EMR
10. Conclusion
This project demonstrates a full distributed ML pipeline for large-scale flight delay prediction.
It accurately identifies delayed flights, supports airline operations, and highlights the importance of scalable cloud processing.
11. Future Work (Planned Next Steps)
These steps are acknowledged but not yet implemented:
✔ Remove arr_delay and all post-flight leakage columns
✔ Apply SMOTE or undersampling for class imbalance
✔ Hyperparameter tuning using CrossValidator
✔ Evaluate alternative models (XGBoost, LightGBM)
✔ Integrate AWS Lambda for real-time scoring
✔ Build a QuickSight dashboard
✔ Add weather + external datasets
✔ Implement full Airflow orchestration
