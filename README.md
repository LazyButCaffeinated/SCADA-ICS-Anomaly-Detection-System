# SCADA/ICS Anomaly Detection System
> AI-powered cyberattack detection on industrial control systems using Random Forest and real-time SIEM integration

---

## Overview
This project implements an end-to-end cyberattack detection pipeline on the **SWaT (Secure Water Treatment) dataset** — a real-world industrial control system (ICS/SCADA) dataset from the Singapore University of Technology and Design (SUTD). The system detects multi-stage cyber-physical attacks across 51 industrial sensors and actuators using a **Random Forest Classifier**, extracts critical sensor importances, and forwards real-time alerts with confidence and severity ratings to a **Splunk Cloud SIEM dashboard** for SOC incident response.

---

## Architecture
```
SWaT ICS Dataset (51 sensors: flow, level, pressure, chemical)
        ↓
Data Preprocessing (cleaning, mean imputation, label encoding)
        ↓
Stratified 80/20 Train-Test Split & StandardScaler Normalization
        ↓
┌──────────────────────────────────────────────┐
│  Random Forest Classifier (Supervised)       │  → 100% Accuracy / 1.00 F1 Score
│  Gini Feature Importance Sensor Analysis     │  → Identifies targeted sensors
└──────────────────────────────────────────────┘
        ↓
Attack Detected (Confidence Scoring & Dynamic Severity Tagging)
        ↓
Splunk HTTP Event Collector (HEC) → SOC SIEM Dashboard
```

---

## Dataset
- **Source:** SWaT (Secure Water Treatment) Dataset — iTrust, Singapore University of Technology and Design (SUTD)
- **Size:** 1,387,098 normal operating rows + 54,621 attack rows
- **Features:** 51 industrial sensors and actuators across 6 physical water treatment stages
- **Monitored Telemetry:** Flow indicators (`FIT101`), level transmitters (`LIT101`), chemical analyzers (`AIT202`), differential pressure indicators (`DPIT301`), motorized valves, and pumps
- **Attack Types:** Sensor spoofing, actuator tampering, false data injection (FDI), and physical state manipulation

---

## Model Performance

| Model | Type | Accuracy | Precision | Recall | F1-Score |
|---|---|---|---|---|---|
| **Random Forest** | Supervised Ensemble | **100%** | **1.00** | **1.00** | **1.00** |

### Why Random Forest for SCADA/ICS?
- **Non-Linear Physical Dynamics:** Industrial processes exhibit complex, non-linear correlations across multi-stage physical cycles (e.g., chemical dosing coupled with tank levels). Random Forest decision trees effectively capture these interactions without assumptions of linearity.
- **Explainable Feature Importance:** Gini importance rankings pinpoint which physical sensors were manipulated during an attack, providing actionable root-cause telemetry to SOC analysts.
- **Calibrated Probabilistic Confidence:** Utilizes `predict_proba` outputs to establish probabilistic confidence thresholds, dynamically routing alerts by severity (`HIGH` vs `MEDIUM`).
- **Variance Mitigation:** Bagging (bootstrap aggregation) prevents overfitting on high-dimensional sensor noise and transient mechanical fluctuations.

---

## Tech Stack
- **ML & Data Processing:** Python, Scikit-Learn, Pandas, NumPy, Matplotlib, Seaborn
- **Environment:** Google Colab / Jupyter
- **SIEM & Incident Response:** Splunk Cloud (HTTP Event Collector / HEC)
- **Data Source:** SWaT ICS Testbed Dataset (iTrust)

---

## Results & Highlights
- Successfully detected **54,621 cyberattack instances** across 5 distinct attack vectors.
- Achieved **100% precision, recall, and F1-score** on the held-out stratified test dataset.
- Computed **sensor feature importance** to isolate critical targets such as tank levels (`LIT101`) and inflow rates (`FIT101`).
- Automated real-time alert dispatch to **Splunk Cloud HEC** with HTTP 200 delivery confirmation.
- Built a SOC-ready alert payload including attack classification, confidence score, severity rating, and live sensor telemetry.

---

## Splunk Telemetry & Alert Structure
Real-time alerts sent to the Splunk HTTP Event Collector adhere to the following SOC schema:
```json
{
  "time": 1715681565,
  "sourcetype": "scada_ids_alert",
  "index": "main",
  "event": {
    "alert_type": "CYBERATTACK DETECTED",
    "severity": "HIGH",
    "confidence": 0.9984,
    "sensors": {
      "LIT101": 817.67,
      "FIT101": 2.49,
      "AIT202": 8.46,
      "DPIT301": 14.6
    },
    "timestamp": "2026-05-14 10:12:45",
    "source": "SWaT_ICS_Anomaly_Detector"
  }
}
```

---

## Setup & Usage

### 1. Clone the repository
```bash
git clone https://github.com/LazyButCaffeinated/SCADA-ICS-Anomaly-Detection-System.git
cd SCADA-ICS-Anomaly-Detection-System
```

### 2. Install dependencies
```bash
pip install scikit-learn pandas numpy matplotlib seaborn requests
```

### 3. Add dataset
Download the SWaT dataset from [Kaggle](https://www.kaggle.com/datasets/vishala28/swat-dataset-secure-water-treatment-system) and place the files:
```
Datasets/
├── normal.csv
└── attack.csv
```

### 4. Configure Splunk Cloud HEC
In `scada.ipynb`, update your Splunk HTTP Event Collector endpoint and authorization token:
```python
SPLUNK_URL   = "https://<your-instance>.splunkcloud.com:8088/services/collector/event"
SPLUNK_TOKEN = "<your-hec-token>"
```

### 5. Run the notebook
Open `scada.ipynb` in Google Colab or your local Jupyter environment and execute all cells sequentially.

---

## Key Concepts Demonstrated
- **Supervised Cyber-Physical Intrusion Detection:** Classifying stealthy sensor tampering and actuator attacks in critical infrastructure.
- **Root-Cause Sensor Attribution:** Leveraging tree-based feature importance to identify which industrial process points were compromised.
- **SIEM Integration:** Bridging operational technology (OT) sensor telemetry with enterprise security operations (Splunk HEC).
- **Industrial Cybersecurity Alignment:** Implementing monitoring principles aligned with the **NIST SP 800-82** Guide to Industrial Control Systems (ICS) Security and the **Purdue Enterprise Reference Architecture**.

---

## Relevance to Industry
Industrial control systems are high-value targets for nation-state threat actors and cyber-sabotage — as evidenced by Stuxnet (2010), the Ukrainian Power Grid attacks (2015/2016), and the Oldsmar Water Treatment facility intrusion (2021). This project addresses critical infrastructure defense by combining machine learning anomaly detection with automated SOC escalation.

---

## Author
**Rajat Bhat**  
M.E. Cyber Security — Manipal School of Information Sciences  
[LinkedIn](https://linkedin.com/in/rajat-bhat-139152341/) | rajatbhat0703@gmail.com
