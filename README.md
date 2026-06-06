# NIFTY-Options-IV-Surface-Reconstruction
# NIFTY Options IV Surface Reconstruction

## Project Overview

This project was developed for the Finance Club, IIT Roorkee Open Project 2026: **Implied Volatility Surface Prediction**.

The objective of the project is to predict missing implied volatility values in NIFTY options market data across different timestamps and strikes. The final output is a Kaggle-compatible `submission.csv` file containing predictions for the missing implied volatility entries.

Implied volatility is a key quantity in options pricing, risk management, hedging, volatility trading, and derivatives market-making. Since implied volatility usually forms a structured surface across strikes and time, this project focuses on reconstructing the missing values using financial intuition rather than treating the data as unrelated tabular cells.

---

## Repository Contents

| File                                                  | Description                                                                                    |
| ----------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `NIFTY Options IV Surface Reconstruction_final.ipynb` | Main notebook containing preprocessing, validation, modelling, and final submission generation |
| `submission.csv`                                      | Final Kaggle submission file                                                                   |
| `filled_dataset.csv`                                  | Dataset after filling missing implied volatility values                                        |
| `v3_validation_report.csv`                            | Internal masked-validation results                                                             |
| `v3_ensemble_weights.csv`                             | Learned ensemble weights for different missing-value situations                                |
| `submission-converter.ipynb`                          | Helper notebook for converting a filled dataset into submission format                         |
| `sandbox_solution.csv`                                | Sample submission / required submission format                                                 |
| `.gitignore`                                          | Prevents restricted/raw dataset files from being pushed                                        |

> Note: The original `dataset.csv` is not included in this public repository because it may be competition-provided data.

---

## Problem Statement

The task is to fill missing implied volatility values in a NIFTY options dataset. The dataset contains option IV values across:

* timestamps,
* strikes,
* call options,
* put options.

The goal is to minimize Mean Squared Error between predicted IV values and hidden ground-truth IV values.

---

## Financial Intuition

The main idea behind the solution is that implied volatility values are not independent random entries. They form a structured volatility surface.

At a fixed timestamp, IV generally varies smoothly across nearby strikes and may show smile/skew-like behaviour. Therefore, a missing IV value can often be estimated from:

* neighbouring strikes at the same timestamp,
* local strike-wise interpolation,
* polynomial smile/skew fitting,
* recent past IV values of the same option contract.

The model handles calls and puts separately because they may have different strike ranges, missingness patterns, and surface behaviour.

---

## Methodology

The final solution uses a **validation-weighted ensemble imputation model**.

It combines the following base methods:

1. **Linear strike-wise interpolation**

   * Stable local interpolation across nearby strikes.

2. **Akima interpolation**

   * Smooth local interpolation that avoids excessive oscillation.

3. **Quadratic polynomial fitting**

   * Captures basic smile/skew-like curvature in the IV surface.

4. **Cubic polynomial fitting**

   * Captures more flexible local curvature.

5. **Past-time fill**

   * Uses only previous timestamp values for the same option contract.
   * This captures short-term IV persistence while avoiding future-time leakage.

The final prediction is a weighted ensemble:

```text
Final IV =
w1 × linear interpolation
+ w2 × Akima interpolation
+ w3 × quadratic polynomial fit
+ w4 × cubic polynomial fit
+ w5 × past-time fill
```

The weights are learned using internal masked validation.

---

## Validation Strategy

To reduce overfitting to the public leaderboard, the model uses internal validation by hiding known IV values and trying to predict them back.

Three validation schemes are used:

1. **Random masking**

   * Randomly hides known IV cells to test general robustness.

2. **Actual-pattern masking**

   * Mimics the actual missingness pattern in the dataset.

3. **Edge-strike masking**

   * Specifically tests difficult extrapolation cases at extreme strikes.

This validation strategy helps evaluate whether the model can generalize to hidden private leaderboard data.

---

## Handling Different Missing-Value Cases

The model assigns each missing value to an adaptive group based on:

* whether the missing strike is an interior strike or edge strike,
* whether the timestamp is close to expiry,
* whether the row has enough observed IV values.

Different groups receive different ensemble weights. This makes the method more robust because IV behaviour can change significantly near expiry or at extreme strikes.

---

## Lookahead Bias Prevention

The model avoids using future timestamp information while predicting missing values.

Same-timestamp strike information is used because it is part of the observed IV surface at that point in time. However, for time-based filling, only past observations are used through forward fill.

This makes the approach safer and more realistic for financial time-series modelling.

---

## Final Result

The final selected submission file is:

```text
submission.csv
```

The best public leaderboard score achieved by the final private-safe version was:

```text
Public MSE: 0.0000378485
```

This submission was selected because it had the best public score among tested variants while also following a robust internal validation strategy.

---

## How to Run

Place the competition-provided `dataset.csv` and `sandbox_solution.csv` files in the same folder as the notebook.

Then run:

```text
NIFTY Options IV Surface Reconstruction_final.ipynb
```

The notebook will generate:

```text
submission.csv
filled_dataset.csv
v3_validation_report.csv
v3_ensemble_weights.csv
```

---

## Requirements

The notebook uses the following Python libraries:

```python
pandas
numpy
scipy
scikit-learn
```

Install dependencies using:

```bash
pip install pandas numpy scipy scikit-learn
```

---

## Key Strengths of the Approach

* Uses financial structure of implied volatility surfaces.
* Handles calls and puts separately.
* Uses same-timestamp strike-wise interpolation.
* Avoids future-time leakage.
* Uses masked validation instead of direct public leaderboard tuning.
* Learns adaptive ensemble weights for different market situations.
* Generates a reproducible final submission file.

---

## Author

Krishna6203

Project: NIFTY Options IV Surface Reconstruction
Finance Club, IIT Roorkee Open Project 2026
