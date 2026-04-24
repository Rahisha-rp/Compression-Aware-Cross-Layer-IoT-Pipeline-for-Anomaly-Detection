Compression-Aware Cross-Layer IoT Pipeline for Anomaly Detection


Overview

This project presents an **end-to-end cross-layer IoT pipeline simulator** that jointly optimizes three pipeline layers typically studied in isolation:

1. **Edge Layer** — Huffman coding compression to reduce sensor payload size
2. **Network Layer** — SimPy-based simulation of MQTT vs. CoAP protocols under realistic packet loss (0%–30%)
3. **Cloud Layer** — PCA dimensionality reduction + weighted ensemble anomaly detection

The pipeline is evaluated on the **UCI Air Quality dataset** and achieves:
- 2.75× compression ratio** (~63% bandwidth reduction)
- F1-score of 0.707** (up from 0.531 baseline — a **33% improvement**)
- Optimal config: CoAP @ 20% packet loss** (composite score: 0.823)

---

## Repository Structure

```
IoT-Pipeline-Simulator/
│
├── Compression-Aware-Cross-Layer-IoT-Pipeline-for-Anomaly-Detection.ipynb   # Main Jupyter notebook (full pipeline)
├── AirQualityUCI.xlsx                   # Dataset file (download separately, see below)
├── README.md                            # This file
---

## Dataset

UCI Air Quality Dataset**

| Property | Details |
|----------|---------|
| Source | UCI Machine Learning Repository |
| Download |  [https://archive.ics.uci.edu/dataset/360/air+quality](https://archive.ics.uci.edu/dataset/360/air+quality) |
| Format | Excel (`.xlsx`) |
| Records | 9,357 hourly observations |
| Features | 15 (4 used: CO, NOx, Temperature, Humidity) |
| Time Period | March 2004 – February 2005 |

**Note:** Download the dataset from the link above and place `AirQualityUCI.xlsx` in your working directory (or Google Drive if using Colab) before running the notebook.

---

## Getting Started

### Prerequisites

```bash
pip install simpy scikit-learn matplotlib seaborn openpyxl xgboost imbalanced-learn pandas numpy
```

### Running on Google Colab (Recommended)

1. Upload `IoT_Pipeline_AirQuality_v6-3.ipynb` to Google Colab
2. Upload `AirQualityUCI.xlsx` to your Google Drive at:
   ```
   MyDrive/air quality/AirQualityUCI.xlsx
   ```
3. Run all cells in order

### Running Locally

1. Clone the repository:
   ```bash
   git clone https://github.com/<your-username>/IoT-Pipeline-Simulator.git
   cd IoT-Pipeline-Simulator
   ```
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Place `AirQualityUCI.xlsx` in the root directory
4. Launch Jupyter:
   ```bash
   jupyter notebook IoT_Pipeline_AirQuality_v6-3.ipynb
   ```

---

## Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     IoT Sensor Nodes (50)                       │
│           CO │ NOx │ Temperature │ Humidity                     │
└──────────────────────────┬──────────────────────────────────────┘
                           │
              ┌────────────▼────────────┐
              │   Stage 1: Compression  │
              │   Huffman Encoding      │
              │   Ratio: ~2.75×         │
              └────────────┬────────────┘
                           │
              ┌────────────▼────────────┐
              │  Stage 2: Network Sim   │
              │  MQTT  vs  CoAP         │
              │  Packet Loss: 0–30%     │
              └────────────┬────────────┘
                           │
              ┌────────────▼────────────┐
              │  Stage 3: Cloud Layer   │
              │  Feature Eng. (138 ft.) │
              │  PCA (136 → 12 dims)    │
              │  Ensemble Detection     │
              └────────────┬────────────┘
                           │
              ┌────────────▼────────────┐
              │  Stage 4: Cross-Layer   │
              │  Composite Optimization │
              │  Best: CoAP @ 20% loss  │
              └─────────────────────────┘
```

---

## Methodology

### Stage 1 — Edge Compression (Huffman Coding)
- Sensor readings quantized into 100 discrete bins
- Huffman tree built from symbol frequencies
- Average compression ratio: **2.75×**
- Payload reduced from 18,652 → 6,827 bytes/cycle

### Stage 2 — Network Simulation (SimPy)
| Protocol | Header | Connection | Retransmission Delay |
|----------|--------|------------|----------------------|
| MQTT | 2 bytes | Persistent (TCP-like) | 0.5 ticks |
| CoAP | 4 bytes | Connectionless (UDP-like) | 2.0 ticks |

- 50 sensors × 500 simulation ticks
- Packet loss rates tested: 0%, 5%, 10%, 20%, 30%

### Stage 3 — Anomaly Detection
- **Feature Engineering:** 4 raw signals → 138 features (rolling stats, lags, diffs, EMAs, cross-sensor interactions)
- **PCA:** 136 → 12 components (retains ~88% variance)
- **Models evaluated:**
  - Isolation Forest (tuned)
  - Local Outlier Factor (n=30)
  - One-Class SVM ← *Best individual model (F1 = 0.707)*
  - PCA Mahalanobis Distance
  - Weighted Ensemble of all 4
  - XGBoost semi-supervised blend

### Stage 4 — Cross-Layer Optimization
Composite Score = `0.5 × F1 + 0.3 × Delivery Rate + 0.2 × Bandwidth Efficiency`

---

## Results

### Compression
| Feature | Compression Ratio | Bandwidth Reduction |
|---------|-------------------|---------------------|
| CO | 2.68× | ~62.7% |
| NOx | 2.71× | ~63.1% |
| Temperature | 2.80× | ~64.3% |
| Humidity | 2.82× | ~64.5% |
| **Average** | **2.75×** | **~63%** |

### Anomaly Detection
| Model | Precision | Recall | F1-Score |
|-------|-----------|--------|----------|
| One-Class SVM | 0.741 | 0.675 | **0.707** |
| Weighted Ensemble | 0.718 | 0.668 | 0.692 |
| XGBoost Blend | 0.724 | 0.672 | 0.697 |
| Isolation Forest | 0.698 | 0.671 | 0.684 |
| LOF | 0.681 | 0.643 | 0.661 |
| **Baseline** | 0.543 | 0.521 | **0.531** |

### Cross-Layer (Best Configurations)
| Protocol | Packet Loss | F1 | Composite Score |
|----------|-------------|-----|-----------------|
| **CoAP** | **20%** | 0.690 | **0.823** ✅ |
| CoAP | 0% | 0.705 | 0.814 |
| MQTT | 0% | 0.703 | 0.811 |

---

## 📉 Ablation Study

| Configuration | F1-Score | Drop vs. Full |
|---------------|----------|----------------|
| Full pipeline | **0.707** | — |
| No feature engineering | 0.679 | −0.028 |
| No PCA | 0.646 | −0.061 |
| Single model (IF only) | 0.656 | −0.051 |
| No XGBoost | 0.692 | −0.015 |

> PCA is the single most impactful component in the pipeline.

---

## Technologies Used

| Category | Tools |
|----------|-------|
| Language | Python 3 |
| Data Processing | pandas, NumPy |
| Machine Learning | scikit-learn, XGBoost |
| Network Simulation | SimPy |
| Visualization | matplotlib, seaborn |
| Compression | heapq, collections (custom Huffman) |
| Environment | Google Colab / Jupyter Notebook |

---

##  References

1. L. Atzori, A. Iera, G. Morabito, "The Internet of Things: A survey," *Computer Networks*, 2010. https://doi.org/10.1016/j.comnet.2010.05.010
2. M. V. Narayana and H. Dakhore, "Investigation of energy cost of data compression algorithms in WSN," *PMC/NCBI*, 2022.
3. F. F. Moghaddam et al., "Power evaluation of IoT application layer protocols," *arXiv*, 2024.
4. E. Achiluzzi et al., "Exploring the use of data-driven approaches for anomaly detection in the IoT environment," *arXiv*, 2023.
5. H. P. Medeiros et al., "Lightweight data compression in WSN using Huffman coding," *Marquette University*, 2014.
6. A. Campazas-Vega et al., "A performance analysis of IoT networking protocols," *Applied Sciences*, 2021.
7. A. Behal et al., "Comparison of MQTT and CoAP using Contiki OS," *ICCCNT*, 2023.
8. Towards Data Science, "Real-time anomaly detection with Python," 2024.

---

## 📄 License

This project was developed for academic purposes at the University of South Dakota. Not intended for commercial use.
