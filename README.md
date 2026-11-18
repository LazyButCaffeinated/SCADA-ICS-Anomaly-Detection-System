# SCADA/ICS Cyberattack Detection & SOC SIEM System
> AI-powered intrusion detection for industrial control systems using regularized Random Forest and real-time Splunk Cloud SIEM integration

---

## 📌 Executive Summary
Industrial Control Systems (ICS) and SCADA environments govern critical physical infrastructure—such as municipal water treatment plants, electrical power grids, and chemical facilities. Traditional IT Intrusion Detection Systems (NIDS) fail to identify **cyber-physical attacks** where attackers spoof sensor signals or manipulate actuator commands using valid industrial protocols.

This project implements an end-to-end cyber-physical intrusion detection pipeline trained and evaluated on the **SWaT (Secure Water Treatment)** testbed dataset from Singapore University of Technology and Design (SUTD). The system employs a regularized **Random Forest Classifier** across **51 industrial sensors and actuators**, extracts root-cause sensor feature importances, and dispatches real-time, confidence-scored alerts via REST API to a **Splunk Cloud SIEM** dashboard.

---

## 🏗️ Architecture

```
                 SWaT Industrial Water Treatment Testbed
          (51 Sensors & Actuators across 6 Physical Stages)
                                 │
                                 ▼
           Data Preprocessing & Anomaly Pipeline
       ├── Whitespace & Schema Sanitization
       ├── Mean Imputation for Missing Telemetry
       ├── Stratified 80/20 Train-Test Split (203,700 / 50,925)
       └── StandardScaler Feature Normalization
                                 │
                                 ▼
         Regularized Random Forest Classifier
       ├── 100 Estimators, max_depth=12, min_samples_split=10
       ├── Anti-Overfitting Tree Pruning (Prevents 100% Data Leakage)
       └── Gini Feature Importance Attribution (LIT101, FIT101, AIT202)
                                 │
                                 ▼
                Confidence Scoring & Severity Engine
       ├── Probabilistic Confidence Scoring via predict_proba
       ├── Dynamic Severity Mapping: HIGH (Confidence > 0.85) | MEDIUM
       └── Structured JSON Event Construction with Sensor Snapshots
                                 │
                                 ▼
                 Splunk Cloud HTTP Event Collector (HEC)
                SOC Incident Response & Alert Dashboard
```

---

## 📊 Dataset: SWaT (Secure Water Treatment)
- **Source:** iTrust Centre for Research in Cyber Security, Singapore University of Technology and Design (SUTD).
- **Scale:** 1,387,098 normal operating rows (11 days of continuous operation) + 54,621 physical attack rows (4 days under active cyber-physical attacks).
- **Process Stages:** 6 coordinated physical stages (P1: Raw water supply & storage, P2: Chemical dosing & pH balancing, P3: Ultrafiltration, P4: Dechlorination, P5: Reverse Osmosis, P6: Backwash cleaning).
- **Monitored Telemetry (51 Features):**
  - **Level Transmitters:** `LIT101`, `LIT301`, `LIT401`
  - **Flow Meters:** `FIT101`, `FIT201`, `FIT301`, `FIT401`, `FIT501`, `FIT601`
  - **Chemical Analyzers:** `AIT201`-`AIT203`, `AIT401`-`AIT402`, `AIT501`-`AIT504`
  - **Differential Pressure:** `DPIT301`, `PIT501`-`PIT503`
  - **Actuators:** Motorized valves (`MV101`, `MV201`, `MV301`) and industrial pumps (`P101`, `P201`, `P301`)
- **Simulated Threat Vectors:** Sensor spoofing, false data injection (FDI), actuator locking, tank overflow attacks, and replay attacks.

---

## 📈 Model Performance & Validation

To build a production-viable model rather than an overfitted laboratory baseline, the Random Forest ensemble was regularized (`max_depth=12`, `min_samples_split=10`, `min_samples_leaf=5`, `max_features='sqrt'`). This constraints the decision trees from memorizing adjacent second-by-second time-series samples and forces the ensemble to learn physical process invariants.

### Classification Report (Held-Out Test Set: 50,925 Samples)

| Class | Precision | Recall | F1-Score | Support |
|---|---|---|---|---|
| **Normal (0)** | 0.9929 | 0.9871 | 0.9900 | 40,001 |
| **Attack (1)** | **0.9537** | **0.9740** | **0.9637** | **10,924** |
| **Accuracy** | — | — | **0.9843 (98.4%)** | **50,925** |
| **Macro Average** | 0.9733 | 0.9806 | 0.9768 | 50,925 |
| **Weighted Average** | 0.9845 | 0.9843 | 0.9843 | 50,925 |

### Confusion Matrix Breakdown

```
                         Predicted Normal    Predicted Attack
Actual Normal (40,001)        39,485                516
Actual Attack (10,924)           284             10,640
```

- **True Positives (Attacks Caught):** **10,640** (97.40% Detection Rate)
- **False Negatives (Missed Stealth Attacks):** **284** (2.60%) — Represents low-rate, micro-drift sensor tampering that momentarily mimics normal mechanical fluctuation.
- **True Negatives (Normal Traffic Verified):** **39,485** (98.71% Specificity)
- **False Positives (False Alarms in SOC):** **516** (1.29% False Alarm Rate) — Critical for avoiding SOC alert fatigue.

---

## 🔍 Root-Cause Analysis: Top Critical SCADA Sensors
Using Gini feature importance, the Random Forest model isolates the top physical variables targeted during multi-stage cyberattacks:

| Rank | Sensor Tag | Gini Importance | Subsystem / Role | Attack Exposure in SWaT |
|:---:|:---:|:---:|---|---|
| **1** | `LIT101` | **0.1842** | Stage 1 Raw Water Tank Level | Primary target for tank overflow and underflow drainage attacks |
| **2** | `FIT101` | **0.1418** | Stage 1 Inflow Water Meter | Spoofed flow readings to hide valve manipulation |
| **3** | `AIT202` | **0.0984** | Stage 2 pH / Conductivity Analyzer | Chemical poisoning attack vector (analogous to Oldsmar 2021) |
| **4** | `DPIT301` | **0.0815** | Stage 3 Differential Pressure Transmitter | Membrane clogging and filtration damage attempts |
| **5** | `LIT301` | **0.0762** | Stage 3 UF Permeate Tank Level | Multi-stage cascade overflow attack |
| **6** | `FIT201` | **0.0649** | Stage 2 Chemical Dosing Flow | Acid / alkali injection rate tampering |
| **7** | `MV101` | **0.0583** | Motorized Valve 101 | Unauthorized valve shut-off / flow starvation |
| **8** | `P101` | **0.0517** | Raw Water Intake Pump | Pump cavitation and dry-run damage attacks |
| **9** | `AIT502` | **0.0471** | Stage 5 Reverse Osmosis Analyzer | High-pressure salt passage manipulation |
| **10** | `FIT401` | **0.0409** | Stage 4 Dechlorination Flow Meter | Chemical neutrality bypass |

---

## 🛡️ Splunk SIEM Integration & Alert Telemetry
Detected attacks are packaged into standardized SOC payloads and transmitted over HTTPS to **Splunk Cloud HTTP Event Collector (HEC)**:

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

### Alert Features:
* **Confidence Scoring:** Derived from `predict_proba` class distribution.
* **Dynamic Severity Tagging:**
  * `HIGH` Severity: Confidence $> 0.85$ (Immediate automated SOC ticket escalation)
  * `MEDIUM` Severity: Confidence $\le 0.85$ (Flagged for Tier-1 analyst triage)
* **Live Physical Snapshots:** Transmits key stage indicators directly in JSON to allow operators to see process anomalies without opening raw SCADA logs.

---


---

## 🚀 Setup & Execution

### 1. Clone Repository
```bash
git clone https://github.com/LazyButCaffeinated/SCADA-ICS-Anomaly-Detection-System.git
cd SCADA-ICS-Anomaly-Detection-System
```

### 2. Install Dependencies
```bash
pip install scikit-learn pandas numpy matplotlib seaborn requests
```

### 3. Dataset Setup
Download the SWaT dataset from [Kaggle](https://www.kaggle.com/datasets/vishala28/swat-dataset-secure-water-treatment-system) and structure:
```
Datasets/
├── normal.csv
└── attack.csv
```

### 4. Configure Splunk Cloud HEC
In `scada.ipynb`, supply your Splunk Cloud endpoint and HEC token:
```python
SPLUNK_URL   = "https://<your-instance>.splunkcloud.com:8088/services/collector/event"
SPLUNK_TOKEN = "<your-hec-token>"
```

### 5. Execute Pipeline
Run `scada.ipynb` in Google Colab or Jupyter to execute the training, validation, feature extraction, and Splunk dispatch.

---

## 👤 Author
**Rajat Bhat**  
M.E. Cyber Security — Manipal School of Information Sciences  
🔗 [LinkedIn Profile](https://linkedin.com/in/rajat-bhat-139152341/) | ✉️ rajatbhat0703@gmail.com
