# FAA Wildlife Strike Analysis
### Predicting Aircraft Damage from Pre-Strike Conditions

---

## Project Overview

This project uses the FAA Wildlife Strike Database to build a machine learning model that predicts whether a wildlife strike will cause aircraft damage — using only information available **before** the strike occurs.

The key challenge was identifying and removing data leakage: post-strike damage flags that made early models look artificially accurate (99%) but were useless in practice.

---

## Dataset

- **Source:** FAA Wildlife Strike Database (`database.csv`)
- **Size:** 174,104 records, 65 original columns
- **Target Variable:** `Aircraft Damage` (0 = No Damage, 1 = Damage)
- **Class Imbalance:** 91.4% No Damage / 8.6% Damage

---

## Problem & Key Finding: Data Leakage

Initial models scored **99% accuracy** by learning from post-strike damage flags (`Wing or Rotor Damage`, `Engine1 Damage`, `Fuselage Damage`, etc.) — columns that are only recorded *because* damage occurred. This is data leakage: the model was reading the answer sheet rather than making real predictions.

**Fix:** All post-strike columns were dropped, leaving only features knowable before a strike:

| Kept (Pre-Strike) | Dropped (Post-Strike / Leakage) |
|---|---|
| Flight Phase | All damage flags (Wing, Engine, Fuselage...) |
| Height | All strike flags (Radome, Windshield...) |
| Speed | Flight Impact |
| Visibility | Fatalities / Injuries |
| Precipitation | Warning Issued |
| Species Name | Engine Ingested |
| Aircraft Mass | Remarks |
| Airport / State / FAA Region | |
| Incident Year / Month | |

---

## Methodology

### 1. Data Cleaning
- Dropped all post-strike leakage columns
- Filled missing values:
  - Categorical columns → mode
  - Numeric columns (`Height`, `Distance`, `Speed`) → median
  - `Species Name` → `'Unknown'`

### 2. Feature Engineering
- Label encoded: `Airport`, `State`, `Species Name`
- One-hot encoded: `FAA Region`, `Flight Phase`, `Precipitation`, `Visibility`, `Species Quantity`

### 3. Handling Class Imbalance
- Applied **SMOTE** (Synthetic Minority Oversampling Technique) to training data
- Before SMOTE: 127,310 vs 11,973
- After SMOTE: 127,310 vs 127,310 (balanced)

### 4. Model Training
- **Algorithm:** Random Forest Classifier
- **Hyperparameters:** `n_estimators=200`, `max_depth=20`, `min_samples_split=10`, `class_weight='balanced'`
- **Train/Test Split:** 80/20

### 5. Threshold Tuning
Default threshold of 0.5 was too conservative for a safety-critical context. Lowered to **0.3** to prioritize catching real damage cases (recall) over minimizing false alarms.

---

## Results

| Threshold | Damage Recall | Damage Precision | Damage F1 | Accuracy |
|---|---|---|---|---|
| 0.50 (default) | 0.32 | 0.41 | 0.36 | 90.1% |
| **0.30 (tuned)** | **0.62** | **0.27** | **0.38** | **82.4%** |

The tuned model catches **62% of real damage cases** from pre-strike information alone — a meaningful result given the inherent difficulty of predicting damage before it occurs.

---

## Top Features (Random Forest Importance)

1. **Aircraft Mass** — heavier aircraft interact differently with bird strikes
2. **Species Name** — large/flocking species cause significantly more damage
3. **Incident Year** — reflects fleet composition and reporting changes over time
4. **Flight Phase (Climb)** — engines at maximum power during climb
5. **Visibility (Night/Day)** — affects pilot ability to see and avoid birds
6. **Height** — altitude at time of strike
7. **Speed** — higher speed = greater impact energy

All top features are genuine pre-strike conditions — confirming the leakage fix worked correctly.

---

## Key Takeaway

> *"Initial models showed 99% accuracy due to data leakage from post-strike damage flags. After removing leakage and retraining with only pre-strike features, the model achieved 62% damage recall — a more honest reflection of what is actually predictable before a strike occurs."*

The drop from 99% to 82% accuracy is not a failure — it is a more truthful model.


