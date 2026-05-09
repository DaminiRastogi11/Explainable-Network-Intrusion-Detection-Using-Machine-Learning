# Explainable Network Intrusion Detection Using Machine Learning

**Course:** CSC532 – Introduction to Machine Learning | University of Illinois Springfield  
**Author:** Damini

---

## Overview

This project builds a binary network intrusion detection system using supervised machine learning on the CIC-IDS2017 dataset, classifying network flows as either benign or attack traffic with high accuracy and full prediction explainability via LIME.

---

## Purpose & Key Functionalities

Traditional rule-based intrusion detection systems fail to catch novel or evolving attacks. This project addresses that gap by training three ML models: Logistic Regression, Random Forest, and XGBoost, on 78 flow-level network features extracted from the CIC-IDS2017 dataset (Canadian Institute for Cybersecurity). The dataset was memory-efficiently loaded from 8 CSV files in 50,000-row chunks, cleaned of duplicates, infinite values, and missing entries, and encoded into binary labels (0 = Benign, 1 = Attack). After hyperparameter tuning with GridSearchCV and stratified cross-validation, XGBoost emerged as the best model with **99.91% accuracy, 99.55% precision, 99.95% recall, 99.75% F1-score, and a perfect 1.0 ROC-AUC**. LIME (Local Interpretable Model-Agnostic Explanations) was applied to the XGBoost model to generate per-prediction feature explanations, making the model transparent and useful for security analysts. The project also includes exploratory data analysis (class distribution, feature histograms, boxplots, and a correlation heatmap) and a fairness discussion tailored to network-traffic classification.

---

## Project Structure

```
├── Damini_Project_Source_code.ipynb   # Main Jupyter notebook (all code)
├── DAMINI_FINAL_PROJECT_REPORT.pdf    # Full written report
├── README.md                          # This file
└── 
```

---

## Dependencies & Installation

This project was developed in **Google Colab** but can be run in any Python 3.8+ environment.

### Required Libraries

Install all dependencies with:

```bash
pip install pandas numpy scikit-learn xgboost lime matplotlib seaborn
```

| Library | Version (recommended) | Purpose |
|---|---|---|
| `pandas` | ≥ 1.3 | Data loading and manipulation |
| `numpy` | ≥ 1.21 | Numerical operations |
| `scikit-learn` | ≥ 1.0 | Logistic Regression, Random Forest, preprocessing, metrics |
| `xgboost` | ≥ 1.5 | XGBoost classifier |
| `lime` | ≥ 0.2 | Local prediction explanations |
| `matplotlib` | ≥ 3.4 | Plotting |
| `seaborn` | ≥ 0.11 | Statistical visualizations |

---

## Dataset Setup

1. Download the **CIC-IDS2017** dataset from the Canadian Institute for Cybersecurity:  
   - go to - https://www.unb.ca/cic/datasets/ids-2017.html
     or 
     [https://www.unb.ca/cic/datasets/ids-2017.html](https://cicresearch.ca/CICDataset/CIC-IDS-2017/)
   - register
   - in Folder: /CIC-IDS-2017/CSVs
   - download MachineLearningCSV.zip

2.  Create a folder named data in the working directory and Unzip and place all 8 CSV files in that. e.g.:

   ```
   /content/data
   ├── Monday-WorkingHours.pcap_ISCX.csv
   ├── Tuesday-WorkingHours.pcap_ISCX.csv
   ├── Wednesday-workingHours.pcap_ISCX.csv
   ├── Thursday-WorkingHours-Morning-WebAttacks.pcap_ISCX.csv
   ├── Thursday-WorkingHours-Afternoon-Infilteration.pcap_ISCX.csv
   ├── Friday-WorkingHours-Morning.pcap_ISCX.csv
   ├── Friday-WorkingHours-Afternoon-PortScan.pcap_ISCX.csv
   └── Friday-WorkingHours-Afternoon-DDos.pcap_ISCX.csv
   ```
3. give this folder path in the notebook.

---

## Running the Code

### Option A — Google Colab (Recommended)

1. Upload `Damini_Project_Source_code.ipynb` to Google Colab.
2. Upload `DATA` in the Colab.
3. Select **Runtime → Run All**.

### Option B — Local Jupyter

```bash
# Clone or download the notebook
jupyter notebook Damini_Project_Source_code.ipynb
```

Update `DATA folder path` to your local dataset path, then run all cells in order.

---

## Reproducing Results

The notebook is fully self-contained and reproducible. Follow these steps:

1. **Data Loading** — The first cells read all 8 CSVs in 50,000-row chunks and sample 10% per chunk. Final shape: **268,048 rows × 78 features**.

2. **Preprocessing** — Duplicate rows (14,736) are removed; infinite and missing values are replaced/dropped; float64 columns are cast to float32 to reduce memory.

3. **Train/Test Split** — An **80/20 stratified split** is applied (`random_state=42`) to preserve class balance.

4. **Hyperparameter Tuning** — GridSearchCV (3-fold, F1-score metric) is run on a 30,000-row sample from the training set. Best parameters:
   - **Logistic Regression:** `C = 10.0`
   - **Random Forest:** `max_depth=None, min_samples_leaf=2, n_estimators=200`
   - **XGBoost:** `colsample_bytree=1.0, learning_rate=0.2, max_depth=4, n_estimators=200, subsample=0.8`

5. **Full Training & Evaluation** — Each tuned model is retrained on all 214,438 training rows and evaluated on the held-out 53,610 test rows.

6. **Expected Results:**

| Model | Accuracy | Precision | Recall | F1-score | ROC-AUC |
|---|---|---|---|---|---|
| Logistic Regression | 0.9331 | 0.7367 | 0.9819 | 0.8418 | 0.9859 |
| Random Forest | 0.9988 | 0.9959 | 0.9975 | 0.9967 | 0.9999 |
| **XGBoost** | **0.9991** | **0.9955** | **0.9995** | **0.9975** | **1.0000** |

7. **LIME Explanations** — The final cells apply LIME to the XGBoost model for one attack sample and one benign sample. The explanations highlight the top 10 features influencing each prediction.

> **Note:** Because a 10% chunk sample is used, exact row counts may vary slightly across runs. Set `random_state=42` wherever sampling is performed to maximize reproducibility.

---

## Key Findings

- **XGBoost** outperformed all other models and missed only **5 attack flows** out of 9,712 in the test set.
- The most discriminative features identified by Random Forest were **Packet Length Std, Max Packet Length, Packet Length Variance**, and **Average Packet Size**.
- LIME confirmed that timing features (Fwd IAT Max, Fwd IAT Mean) and segment-size features (min_seg_size_forward) drive attack predictions, while features like Destination Port and FIN Flag Count are strong benign indicators.

---

## References

- [CIC-IDS2017 Dataset](https://www.unb.ca/cic/datasets/ids-2017.html)
- Sharafaldin et al., "Toward Generating a New Intrusion Detection Dataset," ICISSP 2018.
- Ribeiro et al., "Why Should I Trust You?", NAACL-HLT 2016.
- [CICFlowMeter](https://github.com/ahlashkari/CICFlowMeter)
