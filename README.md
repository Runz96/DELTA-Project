
# Data Processing Pipeline for Delta Project

This document outlines the full workflow for processing raw wearable sensor data from the LifeQ system into digital behavioral phenotypes for post-stroke participants. It covers data download, preprocessing, feature extraction, and summary steps.

---

## 1. Data Download

- Download participant data from the LifeQ shared Google Drive folder.  
- Each participant's data is provided as a zip archive named: `EM-{pid}.zip` (where `{pid}` is a three-digit participant ID).
- Save all zip files to:  
  ```
  OneDrive/Delta Project/Delta Data/DELTA_LifeQ/
  ```

---

## 2. Data Unzipping

- Uncompress each zip file using the provided Jupyter notebook:
  ```
  Data Preprocess/unzip.ipynb
  ```
- This creates individual folders for each participant in the destination directory.

---

## 3. Data Extraction

- Use the notebook:
  ```
  Data Preprocess/extract_data.ipynb
  ```
- Extract the following from each participant folder:
    - **Acceleration data**: `acc.npy`
    - **Photoplethysmography (PPG) data**: `ppg.npy`
- Save these files to:
  ```
  OneDrive/Delta Project/Delta Data/DELTA_ACC_PPG_30secSeg/DELTA_EM-{pid}/
  ```

---

## 4. Human Activity & Sleep Label Prediction

### Activity Recognition
- Run:
  ```
  HAR/transfer_learning_prediction.py
  ```
- Generates predicted activity labels (`acc_pred.npy`) for each 30-second epoch:
    - 0: Light
    - 1: Moderate-vigorous
    - 2: Sedentary
    - 3: Sleep

### Sleep Stage Recognition
- Run:
  ```
  Sleep/WatchSleepNet/run.ipynb
  ```
- Generates sleep stage labels (`sleep_pred.npy`) for main sleep periods:
    - 0: Awake
    - 1: Sleep (NREM)
    - 2: REM

---

## 5. Feature Extraction

- Execute:
  ```
  Feature Extraction/activity_sleep_features.py
  ```
- **Purpose:** Generates behavioral features from processed data.
- **Input:**  
    - `acc_pred.npy`
    - `sleep_pred.npy`
    - `time.csv` (timestamps for each epoch)
- **Output:**  
    - `daily_activity_sleep_features.csv` (per day)
    - `summary_activity_sleep_features.csv` (across all days)

---

## 6. Data Structure

Each participant folder in  
```
OneDrive/Delta Project/Delta Data/DELTA_ACC_PPG_30secSeg/DELTA_EM-{pid}/
```
contains:
- `acc.npy` – Raw acceleration data  
- `acc_pred.npy` – Activity labels  
- `ppg.npy` – Raw PPG data  
- `sleep_pred.npy` – Sleep stage labels  
- `daily_activity_sleep_features.csv` – Daily features  
- `summary_activity_sleep_features.csv` – Summarized features  
- `time.csv` – Timestamps

---

## 7. Feature Extraction and Summarization Details

Raw wearable data (sampled at ~30-second intervals) is processed into digital phenotypes via the following pipeline:

### **A. Epoch-Level Processing**
- Each epoch (row) contains a timestamp, activity label (`acc_pred`), and (for main sleep periods) sleep label (`sleep_pred`).
- Only observed data (with valid timestamps) are analyzed.

### **B. Daily & Intraday Segmentation**
- Data is split by day and within day into four time windows:
    - **Morning:** 06:00–12:00
    - **Afternoon:** 12:00–18:00
    - **Evening:** 18:00–24:00
    - **Night:** 00:00–06:00
- Only periods with valid data are used.

### **C. Feature Calculation**
For each day and time window:
- **General Features:**  
    - Total observed epochs  
    - Wear time (minutes)
- **Physical Activity Features (per class):**  
    - Total minutes  
    - Proportion of wear time  
    - Number & duration of activity bouts  
    - Longest bout  
    - Activity fragmentation (transitions/hour)  
    - Total transitions
- **Sleep Features (main sleep periods):**  
    - Total sleep time, time in bed, sleep efficiency  
    - Sleep onset/offset/midpoint  
    - WASO, awakenings, mean awake bout  
    - REM latency, REM/NREM bouts  
    - Stage transitions, arousals/hour  
    - Stage-specific metrics for full period and night window (00:00–06:00)
- **24-Hour Composition & Coupling:**  
    - Proportion of 24-hour data in each activity class  
    - Activity minutes in the 2 hours before/after main sleep period

### **D. Feature Summarization**
- For each participant, daily features are summarized across all recorded days using:
    - Mean
    - Standard deviation
    - Minimum
    - Maximum
- This produces compact summary variables for further statistical analysis.

---

## 8. Feature List and Definitions

- The full list of 154 extracted features and their definitions is available at:
  ```
  Feature Extraction/feature_definitions.csv
  ```

---

## 9. Example Folder Structure

```
Delta Data/DELTA_ACC_PPG_30secSeg/
└── DELTA_EM-001/
    ├── acc.npy
    ├── acc_pred.npy
    ├── ppg.npy
    ├── sleep_pred.npy
    ├── daily_activity_sleep_features.csv
    ├── summary_activity_sleep_features.csv
    └── time.csv
```

---

## 10. Notes

- Timestamp series may be discontinuous due to non-wear or device issues; all analyses are based on available data only.
- Each step above should be run in the specified order to ensure data integrity and reproducibility.

---

**Contact:**  
For questions or issues, please contact Runze Yan or your project lead.
