# Stochastic Interest Rate Modelling and Prediction

## Finance Club, IIT Roorkee Open Project 2026

This repository contains my submission for the Finance Club, IIT Roorkee Open Project 2026 on **Stochastic Interest Rate Modelling and Prediction**.

The project focuses on implementing, calibrating, and extending the **Cox-Ingersoll-Ross (CIR) short-rate model** using historical yield curve data.

---

## Project Objective

The main objective is to reconstruct the yield curve using only the **3-month yield** as the observable short-rate proxy.

For each test date, the model is allowed to use only:

```text
3M yield
````

and must predict the available yield curve maturities:

```text
6M, 9M, 1Y, 2Y
```

The final model is evaluated using **out-of-sample R²**.

The verification requirement is:

```text
Out-of-sample R² > 0.85
```

---

## Final Result

The final selected model is:

```text
Hybrid CIR + Polynomial 2Y Correction
```

The final out-of-sample R² achieved is approximately:

```text
0.852
```

This exceeds the verification threshold of:

```text
0.85
```

Therefore, the final model satisfies the predictive accuracy criterion.

---

## Project Workflow

The notebook follows the complete project pipeline:

1. Data loading
2. Data preprocessing
3. Yield curve exploratory data analysis
4. Cox-Ingersoll-Ross model theory
5. CIR model implementation
6. CIR parameter calibration
7. Base CIR yield curve reconstruction
8. Model extension experiments
9. Final hybrid model selection
10. Out-of-sample evaluation
11. Critical analysis and limitations

---

## Dataset

The project uses three CSV files provided by Finance Club:

```text
train_data.csv
test_data.csv
test_data_3M.csv
```

The raw dataset is not included in this repository unless explicitly permitted by the project organizers.

To run the notebook, upload the required CSV files to the Colab runtime.

---

## Data Preprocessing

The raw dataset uses zero-coupon maturity labels such as:

```text
ZC025YR, ZC050YR, ZC075YR, ZC100YR, ZC200YR
```

These are renamed into readable maturity labels:

```text
3M, 6M, 9M, 1Y, 2Y
```

The preprocessing steps include:

* Cleaning column names
* Converting date columns to datetime format
* Sorting observations chronologically
* Checking missing values
* Checking yield scale
* Converting yields to decimal format if required
* Creating train/test arrays for modelling

The 3M yield is treated as the short-rate proxy.

---

## Cox-Ingersoll-Ross Model

The CIR model describes the evolution of the instantaneous short rate as:

```text
dr_t = kappa(theta - r_t)dt + sigma sqrt(r_t)dW_t
```

where:

* `r_t` is the short rate
* `kappa` is the speed of mean reversion
* `theta` is the long-run mean rate
* `sigma` is the volatility coefficient
* `W_t` is a Brownian motion

The model is useful because the square-root diffusion term helps keep interest rates non-negative.

The Feller condition is checked:

```text
2 * kappa * theta >= sigma^2
```

---

## CIR Calibration

The CIR parameters are calibrated using the training dataset.

The calibrated parameters are approximately:

```text
kappa = 0.166
theta = 0.033
sigma = 0.014
```

The calibrated parameters satisfy the Feller condition.

The mean-reversion half-life is approximately:

```text
4.17 years
```

This indicates that short-rate shocks are persistent in the dataset.

---

## Base CIR Model Performance

The base CIR model reconstructs the yield curve using only the test-day 3M yield.

The base CIR model achieves an overall out-of-sample R² of approximately:

```text
0.761
```

Maturity-wise performance:

```text
6M  : strong performance
9M  : strong performance
1Y  : good performance
2Y  : weak performance
```

The base CIR model performs well for short maturities but struggles with the 2Y maturity.

This is expected because a one-factor short-rate model has limited ability to capture yield curve slope and curvature.

---

## Model Extensions

Several extensions were tested.

### 1. CIR with Residual Correction

A residual correction model was trained to learn the difference between the actual yield curve and the CIR-implied yield curve.

However, this model performed poorly out of sample, indicating overfitting to the training regime.

### 2. Recent Spread Correction

A recent spread-based model was also tested.

This model predicts each maturity as:

```text
Predicted yield = 3M yield + recent average spread
```

This was interpretable but did not outperform the final hybrid model.

### 3. Hybrid CIR + Polynomial 2Y Correction

The final selected model keeps the base CIR predictions for the maturities where CIR performs well:

```text
6M, 9M, 1Y
```

and replaces only the weak 2Y prediction with a polynomial model trained using only the 3M yield.

This keeps the prediction rule valid because the only test-time input is still the 3M yield.

---

## Final Model

The final model is:

```text
Hybrid CIR + Polynomial 2Y Correction
```

It uses:

```text
Base CIR prediction for 6M, 9M, 1Y
Polynomial 3M-only correction for 2Y
```

Final maturity-wise R² values are approximately:

```text
6M  : 0.991
9M  : 0.954
1Y  : 0.880
2Y  : 0.584
```

Final overall out-of-sample R²:

```text
0.852
```

This passes the verification threshold.

---

## Why the Hybrid Model Works

The base CIR model works well for maturities close to the 3M short-rate proxy.

However, the 2Y yield contains more information about expected future rates and curve slope. A one-factor CIR model cannot fully capture this.

The hybrid model preserves the theoretical CIR structure where it performs well and adds a limited data-driven correction where the model underfits.

This gives better out-of-sample performance while still respecting the rule that only the 3M yield can be used as test input.

---

## Files in This Repository

```text
FinClub-CIR-Yield-Curve-Project/
│
├── FinClub_CIR_Yield_Curve_Project.ipynb
├── README.md
└── data/
    └── README.md
```

---

## How to Run

1. Open the notebook in Google Colab.

2. Upload the required CSV files:

```text
train_data.csv
test_data.csv
test_data_3M.csv
```

3. Run all cells from top to bottom.

4. Check the final verification cell.

The final cell prints:

```text
FINAL OUT-OF-SAMPLE R2
PASSED VERIFICATION CRITERION?
```

Expected final result:

```text
FINAL OUT-OF-SAMPLE R2: approximately 0.852
PASSED VERIFICATION CRITERION? True
```

---

## Requirements

The notebook uses the following Python libraries:

```text
numpy
pandas
matplotlib
scipy
scikit-learn
```

These are available by default in Google Colab.

---

## Critical Analysis

### Feller Condition

The calibrated CIR parameters satisfy the Feller condition:

```text
2 * kappa * theta >= sigma^2
```

This supports the theoretical positivity of the short-rate process.

### Mean Reversion

The mean-reversion half-life is around 4.17 years. This suggests that interest-rate shocks are persistent in the dataset.

### Base CIR Limitations

The base CIR model assumes that the entire yield curve is driven by a single short-rate factor.

This makes it less flexible when the curve changes slope or curvature.

### Extension Limitations

The final polynomial correction improves the 2Y maturity but is still based only on the 3M yield. It cannot fully capture macroeconomic expectations, inflation premia, liquidity effects, or central bank policy changes.

### Real-World Limitations

Real yield curves are affected by multiple factors, including:

* Monetary policy expectations
* Inflation expectations
* Liquidity premia
* Market stress
* Regime changes
* Long-term risk premia

More advanced models such as two-factor CIR, jump-diffusion models, or CIR++ could provide additional flexibility, but they also introduce more calibration complexity.

---

## Conclusion

This project implemented and calibrated the Cox-Ingersoll-Ross short-rate model on yield curve data.

The 3M yield was used as the observable proxy for the instantaneous short rate.

The base CIR model performed well for 6M, 9M, and 1Y maturities but underperformed for the 2Y maturity.

To address this, a hybrid extension was implemented. The final model combines CIR predictions for short maturities with a polynomial 2Y correction trained using only the 3M yield.

The final out-of-sample R² is above 0.85, satisfying the Finance Club verification criterion.

---

## Author

Ruhi Gupta
Engineering Physics
Indian Institute of Technology Roorkee


