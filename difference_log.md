# 📊 UNSW-NB15 IDS: Difference Log & Performance Comparison

This document provides a detailed comparison between the **Baseline Research Paper Approach** and the **Improved Ensemble Stacking System** implemented in this project.

## 📈 Executive Summary

| Metric | Research Paper (Baseline) | Improved System (Ours) | Change |
| :--- | :--- | :--- | :--- |
| **Accuracy** | ~87.00% | **90.10%** | 🟢 **+3.10%** |
| **F1-Score** | ~0.896 | **0.915** | 🟢 **+0.019** |
| **ROC-AUC** | ~0.900 | **0.983** | 🟢 **+0.083** |
| **Methodology** | Single Model (XGB/RF) | **Ensemble + Stacking** | Major Upgrade |
| **Optimization** | Default Hyperparameters | **Optuna Bayesian Tuning** | Significant |

---

## 🏗️ System Workflow

The following diagram illustrates the end-to-end pipeline of our improved IDS, from raw network traffic ingestion to the final optimized decision.

![System Workflow](report/images/workflow_diagram.png)

### Workflow Steps:
1.  **Data Ingestion**: Loading official UNSW-NB15 training and testing CSVs.
2.  **Preprocessing**: Handling missing values, encoding categorical features, and robust scaling.
3.  **Feature Engineering**: Generating throughput, packet symmetry, and timing-based features.
4.  **Resampling**: Balancing classes using SMOTETomek to improve detection on rare attacks.
5.  **Ensemble Layer**: Parallel predictions from tuned XGBoost, LightGBM, and CatBoost models.
6.  **Stacking Layer**: Meta-model learning the optimal combination of base model outputs.
7.  **Threshold Opt**: Fine-tuning the classification boundary for maximum F1-score.

---

## 🔍 Detailed Technical Differences

### 1. Model Architecture
*   **Baseline**: Relied on standalone classifiers (mostly XGBoost or Random Forest) without cross-validation based model combination.
*   **Improvement**: Implemented a **3-Layer Stacking Ensemble** combining:
    *   **XGBoost**: Captures complex non-linear gradient-boosted tree patterns.
    *   **LightGBM**: Utilizes leaf-wise growth for higher efficiency on high-dimensional data.
    *   **CatBoost**: Handles categorical features natively, reducing preprocessing bias.
    *   **Meta-Learner**: A Logistic Regression meta-model that interprets the "confidence" of each base learner.

### 2. Advanced Feature Engineering
*   **Baseline**: Used the default 42 features provided in the dataset.
*   **Improvement**: 
    *   **Network Flow Analysis**: Added `sbytes_per_sec`, `dbytes_per_sec`, and `total_bytes_per_sec`.
    *   **Packet Dynamics**: Introduced `bytes_per_pkt_src`, `pkt_ratio`, and `pkt_diff`.
    *   **Stability Enhancements**: Applied `log1p` transformations to `sbytes`, `dbytes`, and `dur` to compress wide-ranging values.
    *   **Total Features**: Expanded from 42 to **64 features** before selection.

---

## 🔬 Error Analysis with LIME (Local Interpretability)

Beyond global metrics, we utilized **LIME** to investigate specific classification errors. This helps distinguish between malicious intent and "noisy" legitimate traffic.

### 1. False Positive Investigation
*   **Discovery**:Legitimate traffic was occasionally flagged as an attack when `sttl` (Time to Live) values were abnormally high.
*   **Resolution**: By prioritizing state-based features over packet timing, we reduced false alarms in high-concurrency environments.

![LIME False Positive](report/images/lime_false_positive.png)

### 2. False Negative Analysis
*   **Discovery**: Some sophisticated attacks (e.g., Fuzzers) were missed when they mimicked normal "handshake" behavior with very low packet counts (`spkts`).
*   **Resolution**: Our ensemble layer (specifically CatBoost) was tuned to look for categorical anomalies in service protocols, catching these subtle deviations.

![LIME False Negative](report/images/lime_false_negative.png)

---

## 🖼️ Visual Evidence

### Ablation Study (Step-by-Step Improvement)
The chart below illustrates how each stage of our improvement pipeline contributed to the final accuracy.
![Ablation Study](report/images/ablation_study_bars.png)

### Performance Curves
The comparison of ROC and Precision-Recall curves shows that our tuned ensemble (Red) significantly outperforms the baseline (Blue) in both detection reliability and precision.
![ROC PR Curves](report/images/roc_pr_comparison.png)

---

## 🛠️ Reproducibility
All results are reproducible using the `Improved_IDS_UNSW_NB15.ipynb` notebook. The pipeline ensures:
1.  **Strict Temporal Split**: No data leakage.
2.  **Robust Scaling**: Handling of outliers in network traffic.
3.  **Bayesian Optimization**: Automated parameter search for peak performance.
