# 🛡️ SCADA/ICS Cyberattack Detection & SOC SIEM System

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/scikit--learn-F7931E?logo=scikit-learn&logoColor=white" alt="scikit-learn">
  <img src="https://img.shields.io/badge/SIEM-Splunk%20Cloud-black?logo=splunk&logoColor=white" alt="Splunk">
  <img src="https://img.shields.io/badge/Framework-NIST%20SP%20800--82-red" alt="NIST">
  <img src="https://img.shields.io/badge/Threat%20Model-MITRE%20ATT%26CK%20ICS-orange" alt="MITRE ATT&CK ICS">
  <a href="https://colab.research.google.com/github/LazyButCaffeinated/SCADA-ICS-Anomaly-Detection-System/blob/main/scada.ipynb">
    <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab">
  </a>
</p>

> **An end-to-end industrial cybersecurity pipeline that detects multi-stage cyber-physical attacks across 51 SCADA sensors/actuators using a regularized Random Forest classifier, extracts root-cause telemetry, and dispatches real-time, confidence-scored alerts to a Splunk Cloud SIEM.**

---

## 📑 Table of Contents
- [Executive Summary](#-executive-summary)
- [System Architecture](#-system-architecture)
- [Industrial Dataset: SWaT Testbed](#-industrial-dataset-swat-testbed)
- [Model Engineering & Generalization](#-model-engineering--generalization)
- [Empirical Performance & Confusion Matrix](#-empirical-performance--confusion-matrix)
- [Root-Cause Attribution: Top SCADA Sensors](#-root-cause-attribution-top-scada-sensors)
- [SOC Alerting & Splunk SIEM Integration](#-soc-alerting--splunk-siem-integration)
- [MITRE ATT&CK for ICS Threat Mapping](#-mitre-attck-for-ics-threat-mapping)
- [Purdue Model & ICS Standards Compliance](#-purdue-model--ics-standards-compliance)
- [Quickstart & Reproduction Guide](#-quickstart--reproduction-guide)
- [Author & Contact](#-author--contact)

---

## 📌 Executive Summary

Modern industrial facilities (water purification plants, power generation stations, petroleum refineries) rely on Operational Technology (OT) and Supervisory Control and Data Acquisition (SCADA) systems to control physical processes. Traditional IT perimeter defenses (firewalls, signature-based NIDS) cannot detect **semantic cyber-physical attacks** where adversaries inject legitimate-looking protocol commands to manipulate physical setpoints—such as the real-world **Oldsmar Water Treatment poisoning attempt (2021)** and **Stuxnet (2010)**.

This system bridges that gap by monitoring raw, high-frequency **Level 1/2 sensor telemetry**. Trained on the **SWaT (Secure Water Treatment)** benchmark dataset:
- Evaluates **51 continuous physical sensors and discrete actuators** across 6 treatment stages.
- Delivers a production-grade **98.4% test accuracy** and **97.4% attack recall** using regularized tree ensembles.
- Implements tree-based **Gini feature importance attribution** to identify compromised sensor tags (`LIT101`, `FIT101`, `AIT202`).
- Dispatches automated, token-authenticated alerts with severity tagging (`HIGH` vs `MEDIUM`) into **Splunk Cloud via HTTP Event Collector (HEC)** for instant SOC response.

---

## 🏗️ System Architecture

```
SWaT SCADA Dataset (51 Sensors & Actuators)
                     │
                     ▼
  Data Preprocessing & Scaling (StandardScaler)
                     │
                     ▼
   Random Forest Classifier (98.4% Accuracy)
                     │
                     ▼
 Attack Detected ➔ Severity Tagging (HIGH / MEDIUM)
                     │
                     ▼
Splunk Cloud SIEM (Real-Time SOC Alerts via HEC)
```

### Simple 4-Step Pipeline:
1. **Sensor Ingestion:** Reads telemetry from 51 physical sensors and actuators (flow rates, tank levels, pressure, chemical pH).
2. **Preprocessing:** Cleans column names, fills missing values with column means, and scales data with `StandardScaler`.
3. **ML Classification:** Random Forest analyzes the multi-sensor patterns to detect cyber-physical attacks with **98.4% accuracy**.
4. **SOC Incident Response:** Pushes real-time alerts with confidence scores and severity tags (`HIGH` / `MEDIUM`) directly into **Splunk Cloud SIEM**.

---

## 📊 Industrial Dataset: SWaT Testbed

The **Secure Water Treatment (SWaT)** dataset, produced by the **iTrust Centre for Research in Cyber Security at Singapore University of Technology and Design (SUTD)**, is the industry standard benchmark for physical process security research.

| Attribute | Specification |
|---|---|
| **Physical Facility** | 6-Stage Scaled Industrial Water Purification Testbed |
| **Normal Operations** | 1,387,098 rows (11 consecutive days of uncompromised physical operation) |
| **Attack Phase** | 54,621 rows (4 days under 36 systematic multi-stage cyberattacks) |
| **Feature Dimensions** | 51 operational variables (24 analog sensors + 27 digital actuators) |
| **Sampling Rate** | 1 Hz continuous telemetry capture (1 reading per second) |

### Monitored Physical Stages:
1. **P1 (Raw Water Supply & Storage):** Level transmitter `LIT101`, inlet flow meter `FIT101`, motorized valve `MV101`, pump `P101`.
2. **P2 (Chemical Dosing):** Water quality sensors `AIT201`–`AIT203` (pH, conductivity, ORP), chemical injection pumps `P201`–`P206`.
3. **P3 (Ultrafiltration - UF):** UF feed pump `P301`, differential pressure transmitter `DPIT301`, permeate tank level `LIT301`.
4. **P4 (Dechlorination & UV):** UV disinfection `UV401`, dechlorination flow meter `FIT401`, sodium bisulfite dosing.
5. **P5 (Reverse Osmosis - RO):** RO high-pressure pumps `P501`/`P502`, conductivity analyzers `AIT501`–`AIT504`, pressure indicators `PIT501`–`PIT503`.
6. **P6 (Backwash System):** Cleaning flow meter `FIT601`, backwash pumps `P601`/`P602`.

---

## ⚙️ Model Engineering & Generalization

In time-series SCADA datasets, consecutive 1-second records exhibit extreme temporal autocorrelation. Training an unconstrained decision tree allows the model to split on arbitrary second-by-second sensor fluctuations, resulting in a **theoretically misleading 100% test score** that fails completely when encountering novel, out-of-sample attack vectors.

To construct a **generalizable, production-ready intrusion detector**, the Random Forest was explicitly regularized:
```python
rf_model = RandomForestClassifier(
    n_estimators=100,        # 100 bagged decision trees
    max_depth=12,            # Caps tree depth to learn physical process envelopes
    min_samples_split=10,    # Requires 10+ samples to branch (guards micro-noise)
    min_samples_leaf=5,      # Enforces statistical stability at leaf nodes
    max_features='sqrt',     # Sub-samples sqrt(51) features per split
    random_state=42,
    n_jobs=-1
)
```

---

## 📈 Empirical Performance & Confusion Matrix

Evaluated on a **stratified hold-out test set of 50,925 samples** (40,001 Normal, 10,924 Attack instances):

### Detailed Classification Metrics
| Class | Precision | Recall | F1-Score | Support |
|---|---|---|---|---|
| **Normal Operational State (0)** | 0.9929 | 0.9871 | 0.9900 | 40,001 |
| **Active Cyber-Physical Attack (1)** | **0.9537** | **0.9740** | **0.9637** | **10,924** |
| **Overall Accuracy** | — | — | **0.9843 (98.4%)** | **50,925** |
| **Macro Average** | 0.9733 | 0.9806 | 0.9768 | 50,925 |
| **Weighted Average** | 0.9845 | 0.9843 | 0.9843 | 50,925 |

### Confusion Matrix Breakdown
```
                         Predicted Normal    Predicted Attack
Actual Normal (40,001)        39,485                516
Actual Attack (10,924)           284             10,640
```

- **True Positives ($TP = 10,640$):** **97.40% Attack Detection Rate**. The system reliably intercepted 10,640 attack instances.
- **False Negatives ($FN = 284$):** **2.60% Miss Rate**. Confined exclusively to low-rate, stealthy drift manipulations that temporarily stayed within normal hydraulic boundaries.
- **True Negatives ($TN = 39,485$):** **98.71% Specificity**. Clean normal operations are reliably passed without interruption.
- **False Positives ($FP = 516$):** **1.29% False Alarm Rate**. Maintains high signal-to-noise ratio to prevent SOC analyst alarm fatigue.

---

## 🔍 Root-Cause Attribution: Top SCADA Sensors

Using Gini impurity feature importance, the Random Forest model automatically ranks which physical devices deviate most significantly during attacks, providing instant diagnostic context:

| Rank | Tag | Importance | Process Location | Physical Attack Vector |
|:---:|:---:|:---:|---|---|
| **1** | `LIT101` | **0.1842** | Stage 1 Raw Water Tank | Primary target for tank overflow and pump cavitation drainage attacks |
| **2** | `FIT101` | **0.1418** | Stage 1 Inflow Water Meter | Spoofed flow telemetry masking unauthorized valve closure |
| **3** | `AIT202` | **0.0984** | Stage 2 Chemical pH Analyzer | Acid/alkali dosing tampering (chemical sabotage) |
| **4** | `DPIT301` | **0.0815** | Stage 3 UF Differential Pressure | Filter membrane stress and rupture attempts |
| **5** | `LIT301` | **0.0762** | Stage 3 Permeate Tank Level | Multi-tank cascade starvation attack |
| **6** | `FIT201` | **0.0649** | Stage 2 Dosing Flow Meter | Replay attack on chemical feed rates |
| **7** | `MV101` | **0.0583** | Stage 1 Motorized Valve | Physical valve locking preventing water intake |
| **8** | `P101` | **0.0517** | Stage 1 Raw Water Pump | Rapid on/off mechanical fatigue attack |
| **9** | `AIT502` | **0.0471** | Stage 5 RO Conductivity | High-salinity contamination bypass |
| **10** | `FIT401` | **0.0409** | Stage 4 Dechlorination Flow | Chlorine neutralizer feed suppression |

---

## 🛡️ SOC Alerting & Splunk SIEM Integration

When the model detects an anomaly, it extracts probability confidence scores and posts a structured JSON payload to **Splunk Cloud HTTP Event Collector (HEC)**:

### Standardized JSON Incident Payload
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
      "DPIT301": 14.60
    },
    "timestamp": "2026-05-14 10:12:45",
    "source": "SWaT_ICS_Anomaly_Detector"
  }
}
```

### Incident Triaging Logic:
* **Severity `HIGH` (Confidence $> 0.85$):** Multi-sensor correlation indicates active physical compromise. Triggers immediate automated PagerDuty/Jira ticket creation in the enterprise SOC.
* **Severity `MEDIUM` (Confidence $\le 0.85$):** Single-sensor deviation or borderline operational fluctuation. Queued for Tier-1 analyst triage to assess potential sensor drift or calibration error.

### Sample Splunk SPL Queries for SOC Analysts:
```spl
# Attack volume trend by severity
index="main" sourcetype="scada_ids_alert" 
| timechart count by event.severity

# Top manipulated sensors during high-confidence incidents
index="main" sourcetype="scada_ids_alert" event.severity="HIGH"
| stats avg(event.sensors.LIT101) as Avg_LIT101, avg(event.sensors.FIT101) as Avg_FIT101 by event.alert_type
```

---

## 🎯 MITRE ATT&CK for ICS Threat Mapping

| MITRE ATT&CK ID | Technique Name | SWaT Simulation Manifestation | How This System Detects It |
|---|---|---|---|
| **[T0836](https://attack.mitre.org/techniques/T0836/)** | Modify Parameter | Altering chemical dosing thresholds or pressure alarms | Sensor deviation in `AIT202` and `DPIT301` outside physical equilibrium |
| **[T0855](https://attack.mitre.org/techniques/T0855/)** | Unauthorized Command Message | Sending forged Modbus/CIP write commands to valves | Inconsistency between valve state (`MV101`) and expected tank rise rate (`LIT101`) |
| **[T0837](https://attack.mitre.org/techniques/T0837/)** | Loss of Protection | Forcing inlet valves open during high tank levels | Level transmitter `LIT101` breaching hydraulic safety boundaries |
| **[T0888](https://attack.mitre.org/techniques/T0888/)** | False Data Injection (FDI) | Spoofing sensor values to mask pump dry-run conditions | Multivariate cross-sensor correlation catches decoupled telemetry |

---

## 🏛️ Purdue Model & ICS Standards Compliance

- **Purdue Model Alignment:**
  - **Level 0/1 (Process & Control):** Physical water tanks, valves, and PLCs generating raw telemetry.
  - **Level 2 (Supervisory):** Industrial PC / SCADA gateway hosting this ML inference pipeline.
  - **Level 3.5 (Industrial DMZ):** Token-authenticated, encrypted HTTPS forwarding across the OT/IT boundary.
  - **Level 4/5 (Enterprise):** Central enterprise SOC monitoring Splunk dashboards.
- **NIST SP 800-82 (Rev 3) Compliance:** Aligns with NIST recommendations for continuous physical process anomaly detection and OT-to-IT incident escalation.

---

## 🚀 Quickstart & Reproduction Guide

### 1. Clone the Repository
```bash
git clone https://github.com/LazyButCaffeinated/SCADA-ICS-Anomaly-Detection-System.git
cd SCADA-ICS-Anomaly-Detection-System
```

### 2. Install Required Dependencies
```bash
pip install scikit-learn pandas numpy matplotlib seaborn requests
```

### 3. Obtain Dataset
Download the SWaT CSV dataset from [Kaggle](https://www.kaggle.com/datasets/vishala28/swat-dataset-secure-water-treatment-system) and place files in:
```
Datasets/
├── normal.csv
└── attack.csv
```

### 4. Configure Splunk Cloud HEC
Open `scada.ipynb` and enter your Splunk Cloud instance URL and HEC token:
```python
SPLUNK_URL   = "https://<your-instance>.splunkcloud.com:8088/services/collector/event"
SPLUNK_TOKEN = "<your-hec-token>"
```

### 5. Run Training & Inference
Execute the notebook in **Google Colab** or **Jupyter**:
```bash
jupyter notebook scada.ipynb
```

---

## 👤 Author & Contact

**Rajat Bhat**  
*M.E. Cyber Security — Manipal School of Information Sciences*  
- 🔗 **LinkedIn:** [linkedin.com/in/rajat-bhat-139152341](https://linkedin.com/in/rajat-bhat-139152341/)  
- ✉️ **Email:** rajatbhat0703@gmail.com  
- 🐙 **GitHub:** [@LazyButCaffeinated](https://github.com/LazyButCaffeinated)
