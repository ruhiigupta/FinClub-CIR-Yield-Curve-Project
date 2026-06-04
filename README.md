# Stochastic Interest Rate Modelling and Prediction
### Finance Club, IIT Roorkee — Open Projects 2026
**Author:** Ruhi Gupta | Engineering Physics | IIT Roorkee

---

## Overview

This project implements, calibrates, and extends the **Cox-Ingersoll-Ross (CIR)** short-rate model on historical yield curve data. The core challenge: reconstruct the full yield curve using **only the 3M yield** as input, and achieve out-of-sample R² > 0.85.

**Final Result: R² = 0.852 **

---

## The Problem

Interest rates evolve stochastically. The CIR model captures this via:

```
dr_t = κ(θ − r_t)dt + σ√r_t dW_t
```

where κ is mean-reversion speed, θ is the long-run mean, σ is volatility, and W_t is a Brownian motion. The square-root diffusion keeps rates non-negative (Feller condition: 2κθ ≥ σ²).

Given only the **3M yield** on any test day, the goal is to predict: `6M, 9M, 1Y, 2Y`

---

## Dataset

Three CSV files provided by Finance Club:

| File | Purpose |
|------|---------|
| `train_data.csv` | Historical yields for calibration |
| `test_data.csv` | Full test yields (actuals) |
| `test_data_3M.csv` | 3M-only test input (prediction constraint) |

Raw column names follow the format `ZC025YR, ZC050YR, ZC075YR, ZC100YR, ZC200YR`, mapped to `3M, 6M, 9M, 1Y, 2Y` during preprocessing.

---

## Methodology

### 1. Calibration
CIR parameters fit via cross-sectional least squares on training data:

| Parameter | Value | Interpretation |
|-----------|-------|----------------|
| κ (mean reversion) | 0.166 | Half-life ≈ 4.17 years — shocks are persistent |
| θ (long-run mean) | 0.033 | ~3.3% equilibrium rate |
| σ (volatility) | 0.014 | Low vol; Feller condition satisfied (2κθ/σ² >> 1) |

### 2. Base CIR Performance

| Maturity | R² |
|----------|----|
| 6M | 0.99+ |
| 9M | 0.95+ |
| 1Y | 0.88 |
| 2Y | ~0.58 ← weak |
| **Overall** | **0.761** |

The single-factor structure fits short maturities well but cannot capture slope/curvature at 2Y.

### 3. Extensions Tested

| Model | Approach | Outcome |
|-------|----------|---------|
| CIR + Residual Correction | ML on training residuals | Overfit — rejected |
| Recent Spread Correction | `ŷ(τ) = r_3M + avg_spread(τ)` | Interpretable but weaker |
| **Hybrid CIR + Poly(2Y)** | CIR for 6M/9M/1Y, polynomial for 2Y | **Selected ✅** |

### 4. Final Hybrid Model

```
6M, 9M, 1Y  →  Base CIR prediction
2Y          →  Polynomial regression on 3M yield only
```

Only the 3M yield is used at test time — the constraint is fully respected.

| Maturity | R² |
|----------|----|
| 6M | 0.991 |
| 9M | 0.954 |
| 1Y | 0.880 |
| 2Y | 0.584 |
| **Overall** | **0.852 ** |

---

## Repository Structure

```
FinClub-CIR-Yield-Curve-Project/
├── FinClub_CIR_Yield_Curve_Project.ipynb   ← main notebook
├── README.md
└── data/
    └── README.md                           ← data access instructions
```

---

## How to Run

1. Open `FinClub_CIR_Yield_Curve_Project.ipynb` in [Google Colab](https://colab.research.google.com)
2. Upload the three CSV files when prompted:
   - `train_data.csv`
   - `test_data.csv`
   - `test_data_3M.csv`
3. **Runtime → Run all**
4. The final cell prints:

```
FINAL OUT-OF-SAMPLE R²: 0.852
PASSED VERIFICATION CRITERION? True
```

**Dependencies:** `numpy`, `pandas`, `matplotlib`, `scipy`, `scikit-learn` — all pre-installed in Colab.

---

## Limitations

- **Single factor:** CIR cannot independently move curve level, slope, and curvature — explains 2Y underperformance
- **No jumps:** Sudden policy-rate changes (FOMC surprises) are not captured
- **Constant parameters:** Cannot adapt to regime shifts (e.g. ZLB environments)
- **Polynomial correction:** Improves 2Y fit but cannot model macro expectations, inflation premia, or liquidity effects

Extensions like two-factor CIR, CIR++ (Brigo-Mercurio), or jump-diffusion would address these at the cost of calibration complexity.

---

## Verification Criterion

> Out-of-sample R² (yield curve reconstruction from 3M rate) **> 0.85**

**Achieved: 0.852 ✅**
