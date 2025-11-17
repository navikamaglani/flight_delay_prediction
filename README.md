Flight Delay Prediction — Distributed Machine Learning Pipeline (PySpark + AWS)
1. Introduction
Flight delays significantly affect airline operations, airport efficiency, and passenger experience.
This project explores a scalable, cloud-based machine learning workflow to predict flight arrival delays using PySpark and AWS.
The goal is to demonstrate:
How large flight datasets can be processed using PySpark
How delay prediction can be modeled using MLlib
How cloud storage (S3) and Athena improve querying and ETL
How model evaluation highlights operational trade-offs
This work is based entirely on one Jupyter Notebook, which includes the complete workflow: EDA → preprocessing → modeling → evaluation.
2. Dataset Overview
The dataset contains U.S. domestic flight performance records with timestamps, carriers, routes, and detailed delay reasons.
Source: Public airline delay data
Storage (original project setup): S3 → Parquet → Athena (conceptual)
Feature Categories
Pre-Flight Features (Valid Predictors)
These are known before departure.
Column	Description
month	Month of flight (1–12)
carrier	Airline carrier code
Route	Origin–Destination pair
Post-Flight Features (Leakage — Cannot Be Used for Prediction)
These are only known after landing.
Column	Description
arr_delay	Arrival delay in minutes
carrier_delay	Delay caused by carrier
weather_delay	Weather delay
nas_delay	NAS delay
security_delay	Security delay
late_aircraft_delay	Connecting aircraft delay
Delayed_Arrival	Target variable
Dataset size: Millions of rows (Jan 2020 – Feb 2025)
Transparency Note
During early experimentation, post-flight delay columns were included in the model to study correlations and feature importance.
These create label leakage and are not valid for real pre-flight prediction.
The EDA and model results using these columns are intentionally kept in the notebook for transparency and learning purposes.
A production-ready version of the pipeline would exclude all leakage variables.
3. Data Preprocessing
Performed within the notebook using PySpark:
Converted string delay columns → numeric
Filled missing values with safe defaults
Encoded categorical fields using StringIndexer
Assembled final feature vector using VectorAssembler
4. Exploratory Data Analysis (EDA)
Key findings:
Major delay contributors: late aircraft, NAS, carrier
Strong class imbalance
arr_delay highly correlated with delay status (leakage)
Delay patterns vary by route and carrier
5. Machine Learning Model
Model: RandomForestClassifier from PySpark MLlib
Chosen for:
Handling large datasets
Capturing non-linear relationships
Providing feature importance
Experimental Pipeline Steps:
Encode categorical features
Assemble all features
Train Random Forest
Evaluate using confusion matrix and recall
6. Model Performance (Experimental)
Confusion Matrix
Predicted On-Time (0)	Predicted Delayed (1)
Actual On-Time (0)	10,051	8,330
Actual Delayed (1)	2,455	28,258
Interpretation:
The model shows high recall for delayed flights.
False positives exist, but in airline operations, overpredicting delays is safer than missing them.
7. Feature Importance (Experimental)
These include leakage variables and are for analysis only.
Feature	Importance (%)
arr_delay	39.0
nas_delay	19.1
late_aircraft_delay	16.1
carrier_delay	15.0
Route_indexed	4.8
8. Cloud Architecture (Conceptual)
Data Flow
Raw CSV → S3 → PySpark (EMR) → Parquet → Athena → ML model
Scalable Architecture (Future)
Airflow → EMR jobs → Model Training → Batch/Real-Time Scoring
9. Challenges & Solutions
Challenge	Solution
Class imbalance	Use recall, F1; consider SMOTE/undersampling
Label leakage	Identified; to be removed in future iteration
High-cardinality categories	Used handleInvalid="keep" and increased maxBins
Large dataset	Used distributed PySpark processing
10. Conclusion
This project demonstrates how PySpark and AWS can be combined to analyze and model large-scale flight delay data.
While the experimental model includes leakage variables, the notebook provides a clear path toward a production-grade pre-flight prediction system.
11. Future Work
Planned but not completed in current notebook:
Remove leakage variables from all modeling
Address imbalance with SMOTE/undersampling
Tune model using CrossValidator
Train XGBoost/LightGBM for comparison
Build real-time inference using AWS Lambda
Add weather and external features
Add Airflow orchestration
