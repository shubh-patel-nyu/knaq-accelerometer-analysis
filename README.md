# Industrial Accelerometer Analysis & Predictive Maintenance
**Shubh Patel — NYU Tandon School of Engineering**

A end-to-end data analysis pipeline built on 52M+ accelerometer observations from an industrial machine across an 18-hour operating cycle. This project covers data ingestion, signal processing, anomaly detection, and predictive maintenance feature engineering — submitted as a take-home assessment for Knaq (AI/ML Intern role).

---

## Project Overview

Industrial machines fail in patterns. The goal of this project is to find those patterns before failure occurs — turning raw sensor data into actionable maintenance intelligence.

Given a single 18.28-hour recording from a 3-axis accelerometer (800 Hz sample rate), this analysis:
- Cleans and validates 52M+ raw sensor readings
- Characterizes the machine's normal operating signature
- Detects and classifies 311 transient shock events
- Identifies three distinct anomaly types with direct maintenance implications
- Proposes five predictive monitoring features for a production ML model

---

## Dataset

| Property | Value |
|----------|-------|
| File | `iov.log_2025_10_26.log` |
| Recording period | Oct 25–26, 2025 |
| Duration | 18.28 hours |
| Sample rate | 800 Hz |
| Total rows | 52,667,071 |
| Valid rows | 52,660,592 |
| Corrupt rows | 6,479 (0.0123%) |

> **Note:** Raw `.log` files are not included in this repo due to file size. The notebook is fully documented and reproducible with equivalent accelerometer data.

---

## Analysis Pipeline

### 1. Data Loading & Cleaning
- Custom parser handles files with or without text headers
- Detects and separates corrupt rows (single `.` heartbeat markers from acquisition pipeline)
- Converts Unix timestamps to human-readable datetime
- Validates effective sample rate against expected 800 Hz

### 2. Exploratory Analysis
- Per-axis statistics (mean, std, skewness, kurtosis)
- Sensor orientation identified from gravity alignment: X = vertical, Z = primary vibration axis
- Vector magnitude computed: `sqrt(x² + y² + z²)`
- Downsampled visualization of 18-hour signal (1 point/second for plotting)

### 3. Stationary Period Detection
- Rolling standard deviation (1-second windows) applied to magnitude
- Bimodal threshold of `std < 0.5 m/s²` cleanly separates active vs stationary states
- 7 significant stationary periods identified including:
  - 33-minute mid-session shutdown
  - 295-minute final shutdown (end-of-shift behavior)

### 4. Frequency Domain Analysis (FFT)
- FFT applied to first 10 minutes of active operation (480,000 samples)
- Dominant frequencies identified:
  - **~24 Hz** — primary operating frequency (fundamental)
  - **~52 Hz** — secondary dominant (mechanical harmonic)
  - **~8.22 Hz** — structural resonance of mounting
- Classic rotating machinery signature — stable baseline for future drift detection

### 5. Anomaly Detection
- **Z-score outliers:** 31,502 samples beyond ±5 standard deviations (0.0598%)
- **Transient events:** 311 shock events above 20.0 m/s² (9.2σ threshold)
- Events grouped with 1-second minimum gap between separate incidents

---

## Key Findings

### Three Distinct Anomaly Types

| Type | Events | Characteristics | Maintenance Implication |
|------|--------|-----------------|------------------------|
| Extreme magnitude shocks | Events 3, 10 (~29 m/s²) | Instantaneous high-energy impacts | Highest immediate risk of mechanical damage |
| Y-axis suppressed impacts | Events 6, 7 (Y < 2 m/s²) | Energy confined to XZ plane only | Possible misalignment or developing fault in force transmission path |
| Recurring 235ms bursts | Events 2, 4, 5, 9 | Identical duration, repeating pattern | Specific component under repeated stress (braking or latching mechanism) |

### Universal Finding
Every single transient event is **Z-axis dominant** — confirming horizontal impacts are the primary failure risk vector for this machine.

### Temporal Clustering
- Events 4, 5, 6 occur within a **21-minute window**
- Events 7, 8, 9 occur within a **15-minute window**
- Suggests the machine enters periodic high-stress operational phases

---

## Visualizations

| Plot | Description |
|------|-------------|
| `time_domain.png` | All three axes + magnitude over 18 hours |
| `stationary_detection.png` | Rolling mean and std used to detect machine state |
| `active_vs_stationary.png` | Full timeline shaded by machine state |
| `frequency_domain.png` | FFT spectrum per axis (0–100 Hz) |
| `outliers.png` | Magnitude timeline with z-score outliers highlighted |
| `top10_events.png` | Detailed 4-second window around each top 10 event |
| `event_summary.png` | Three-panel summary: timeline, severity scatter, Y-axis bar chart |

---

## Recommended Predictive Features

For a production predictive maintenance model, these five features provide the highest signal:

1. **Rolling Z-axis standard deviation** — most sensitive indicator of abnormal vibration
2. **Magnitude spike frequency** — count of events exceeding 20 m/s² per hour as trend metric
3. **Y-axis suppression flag** — binary flag for impacts where Peak Y < 2.0 m/s²
4. **Frequency drift monitor** — track shifts in the 24 Hz fundamental over time
5. **Duration-classified events** — separate instantaneous shocks from 235ms bursts as distinct failure modes

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Python 3 | Core language |
| pandas | Data loading, cleaning, time-series manipulation |
| numpy | Array operations, FFT, statistical computations |
| matplotlib | All visualizations |
| scipy | Signal processing support |

---

## How to Run

```bash
# Clone the repo
git clone https://github.com/shubh-patel-nyu/knaq-accelerometer-analysis.git
cd knaq-accelerometer-analysis

# Install dependencies
pip install pandas numpy matplotlib scipy

# Open the notebook
jupyter notebook knaq_analysis.ipynb
```

> Replace the `.log` file path in the first cell with your own accelerometer data file.

---

## About

This analysis was completed as a take-home technical assessment for Knaq (AI/ML Intern). The project cleared two interview rounds.

**Shubh Patel**
MS Management of Technology — NYU Tandon School of Engineering
[GitHub](https://github.com/shubh-patel-nyu) | [LinkedIn](https://linkedin.com/in/your-linkedin-here)