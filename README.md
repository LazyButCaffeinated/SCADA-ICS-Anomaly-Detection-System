# SCADA/ICS Anomaly Detection System
> AI-powered cyberattack detection on industrial control systems using machine learning and real-time SIEM integration

---

## Overview
This project implements an end-to-end anomaly detection pipeline on the **SWaT (Secure Water Treatment) dataset** — a real industrial control system dataset from Singapore University of Technology (SUTD). The system detects cyberattacks on SCADA sensor readings using machine learning and forwards real-time alerts to a **Splunk SIEM dashboard** for SOC-style incident response.

---

## Architecture
```
SWaT ICS Dataset (51 sensors)
        ↓
Preprocessing (scaling, null handling, sliding window)
        ↓
┌─────────────────────────────────┐
│  Isolation Forest (Unsupervised)│  → 82% F1 Score
│  Random Forest   (Supervised)   │  → 100% Accuracy
└─────────────────────────────────┘
        ↓
Attack Detected → FastAPI Endpoint
        ↓
Splunk HEC → SIEM Dashboard (severity, confidence, sensor ID)
```

---

## Dataset
- **Source:** SWaT Dataset — iTrust, Singapore University of Technology
- **Size:** 1,387,098 normal rows + 54,621 attack rows
- **Features:** 51 industrial sensors (flow, pressure, water level, chemical readings)
- **Attack Types:** Sensor spoofing, actuator attacks, replay attacks, false data injection

---

## Models

| Model | Type | Accuracy | F1 Score |
|---|---|---|---|
| Isolation Forest | Unsupervised | 82% | 0.81 |
| Random Forest | Supervised | 100% | 1.00 |

**Why two models?**
- **Isolation Forest** simulates real-world scenarios where labeled attack data is unavailable — learns only from normal behavior
- **Random Forest** demonstrates performance ceiling when labeled data exists — used for production alerting

---

## Tech Stack
- **ML:** Python, Scikit-Learn, Pandas, NumPy, Matplotlib, Seaborn
- **Environment:** Google Colab
- **SIEM:** Splunk Cloud (HTTP Event Collector)
- **Data:** SWaT ICS Dataset

---

## Results
- Detected **54,621 cyberattack instances** across 5 attack categories
- Achieved **100% precision and recall** with Random Forest on held-out test set
- Forwarded real-time attack alerts to Splunk with **severity tagging** (HIGH/MEDIUM) and **confidence scoring**
- Built SOC-ready dashboard with total attack count, severity breakdown, and confidence trends

---

## Splunk Dashboard
Alerts sent to Splunk include:
```json
{
  "alert_type"  : "CYBERATTACK DETECTED",
  "severity"    : "HIGH",
  "confidence"  : 0.99,
  "sensors"     : {
    "LIT101"  : 817.67,
    "FIT101"  : 2.49,
    "AIT202"  : 8.46,
    "DPIT301" : 14.6
  },
  "timestamp"   : "2026-05-14 10:12:45",
  "source"      : "SWaT_ICS_Anomaly_Detector"
}
```

---

## Setup & Usage

### 1. Clone the repo
```bash
git clone https://github.com/yourusername/scada-ids.git
cd scada-ids
```

### 2. Install dependencies
```bash
pip install scikit-learn pandas numpy matplotlib seaborn requests
```

### 3. Add dataset
Download SWaT dataset from [Kaggle](https://www.kaggle.com) and place:
```
data/
├── normal.csv
└── attack.csv
```

### 4. Configure Splunk
```python
SPLUNK_URL   = "https://your-instance.splunkcloud.com:8088/services/collector/event"
SPLUNK_TOKEN = "your-hec-token"
```

### 5. Run the notebook
Open `scada_ids.ipynb` in Google Colab and run all cells.

---

## Key Concepts Demonstrated
- **Unsupervised vs Supervised anomaly detection** — practical comparison on real ICS data
- **Class imbalance handling** — normal vs attack data distribution
- **SIEM integration** — real-time alert forwarding via Splunk HEC
- **Industrial cybersecurity** — OT/ICS threat detection aligned with NIST ICS security framework

---

## Relevance to Industry
Industrial control systems are a prime target for cyberattacks — Stuxnet (2010), Ukraine Power Grid (2015), and Oldsmar Water Plant (2021) demonstrate the real-world impact of ICS vulnerabilities. This project addresses that threat using modern ML-based anomaly detection directly applicable to manufacturing, energy, and water treatment environments.

---

## Author
**Rajat Bhat**
M.E. Cyber Security — Manipal School of Information Sciences
[LinkedIn](https://linkedin.com/in/rajat-bhat-139152341/) | rajatbhat0703@gmail.com
