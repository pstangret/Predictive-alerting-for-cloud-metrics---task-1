# Predictive-alerting-for-cloud-metrics---task-1
Task 1 - to have results just run main.py

## Predictive Alerting for Cloud Metrics
# Project Overview
This repository contains a prototype for a predictive alerting system designed to forecast incidents in cloud services based on historical metric data. The goal of this project is to anticipate system anomalies or threshold breaches before they occur, providing early warnings to maintain system reliability.

This solution frames the time-series forecasting problem as a supervised classification task using a sliding-window approach.

# Problem Formulation
Cloud metrics are inherently noisy, non-stationary, and prone to sudden regime shifts. To handle this, the continuous time series is converted into discrete training examples using a sliding window:
- Lookback Window (W): Set to 60 time steps (e.g., representing 60 minutes of historical data). These serve as the feature set (X).
- Prediction Horizon (H): Set to 15 time steps. The model predicts whether at least one incident will occur within this future window. This serves as our binary target (y).
By predicting a future window rather than a single exact future point, the system is more robust to slight timing variances in real-world incidents.

# Dataset
As permitted by the task guidelines, this prototype utilizes a synthetic time-series dataset.
- Baseline: The normal operating metric is simulated using a sinusoidal wave (representing daily seasonality) combined with Gaussian noise.
- Incidents: Anomalies are injected as sudden, sustained spikes lasting between 10 and 30 time steps. These intervals are labeled as 1 (incident), while normal operation is labeled as 0.

# Model Selection
For this implementation, a Random Forest Classifier was chosen over traditional statistical methods (like ARIMA) or deep learning approaches (like LSTMs).
Why Random Forest?
1. Non-linear Relationships: It easily captures non-linear patterns and sudden shifts in the sliding W-window.
2. Robust to Noise: Tree ensembles handle noisy cloud metrics well without overfitting to minor fluctuations.
3. Probability Outputs: Random Forest natively outputs class probabilities via .predict_proba(), which is critical for manually tuning alert thresholds.
4. Class Imbalance: It supports class weighting (class_weight='balanced'), ensuring the model pays sufficient attention to the minority "incident" class.

# Evaluation Setup & Alert Thresholds
Standard accuracy is a poor metric for incident prediction due to extreme class imbalance (normal operation vastly outnumbers incidents). Furthermore, time-series data requires chronological splitting (80% train / 20% test) to prevent data leakage from the future into the past.

The evaluation focuses entirely on the Precision-Recall trade-off:
- The system requirements aim for capturing roughly 80% of real incidents (Recall = 0.80).
- Using the Precision-Recall curve, we identify the exact decision boundary (probability threshold) that yields this ~80% recall on the held-out test set.
- Instead of the default 0.5 probability threshold, the model triggers an alert only when the predicted risk exceeds this custom, data-driven threshold, balancing the desired detection rate against a manageable false-positive rate.

# Limitations & Real-World Adaptation
While this prototype demonstrates the core predictive alerting logic, a production environment introduces additional complexities:
- Multi-Variate Data: This prototype uses a single metric. In reality, incidents correlate across multiple metrics (e.g., CPU spikes combined with dropping network I/O). The sliding window approach can be easily adapted to accept an N-dimensional W-window for multi-variate modeling.
- Concept Drift & CI/CD for ML: Cloud workloads change. As suggested in the project brief, a real-world AWS architecture would require two Lambda functions:
  1. Training Pipeline: A scheduled Lambda (e.g., daily) that fetches recent CloudWatch metrics, retrains the Random Forest model to adapt to new baseline behaviors, and saves the artifact to S3.
  2. Inference Pipeline: A high-frequency Lambda (e.g., every minute) that loads the latest model from S3, ingests the last 60 minutes of metrics, and triggers a PagerDuty/Slack alert if the custom probability threshold is breached.
