# Instacart A/B Test: Does a Discount Badge Increase Reorder Rates?

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-8CAAE6?style=for-the-badge&logo=scipy&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)

**Can a discount badge drive more reorders — and is the effect real enough to ship?**  
A full end-to-end A/B testing pipeline using Instacart's public dataset (3.4M orders, 200K users).

---

## ❌ Result: Do Not Ship

> The observed +0.17% uplift has a 24% chance of being pure random noise. With 103K users per group — 18x the required sample size — this non-significance is a genuine finding, not an underpowered test.

| Metric | Value |
|--------|-------|
| Control reorder rate | 43.19% |
| Variant reorder rate | 43.26% |
| Uplift | +0.17% |
| 95% Bootstrap CI | [-0.110%, +0.251%] |
| Mann-Whitney p-value | 0.237 (not significant) |
| Cohen's d | 0.0034 (negligible) |
| SRM check | PASS (p=0.998) |
| Pseudo R² (logistic) | 0.3325 |
| **Final decision** | **Do Not Ship** |

---

## 🧠 The Business Question

Food delivery and grocery platforms live and die by **retention**. Getting a user to reorder is significantly cheaper than acquiring a new one. I simulated a real product experiment:

> *If we show users a discount badge on product cards, does it meaningfully increase reorder rates — and does the revenue justify the discount cost?*

---

## 🔬 Full Analysis Pipeline

```
Step  Task                                              Result
----  ----                                              ------
1     Load data via kagglehub (no manual download)
2     EDA — order patterns, reorder behavior            Day-30 data cap identified
3     SRM check — validate randomization                PASS (p=0.998)
4     Normality check — Shapiro-Wilk + QQ plots         NOT normal -> zero-inflated
5     Primary test — Mann-Whitney U                     p=0.237 (not significant)
6     Secondary test — t-test                           p=0.222 (not significant)
7     Cohen's d — practical effect size                 0.0034 (negligible)
8     Bootstrap CI (10,000 resamples)                   [-0.11%, +0.25%] crosses zero
9     Power analysis                                    Well-powered (103K >> 5.5K needed)
10    Multiple testing correction — Bonferroni + BH     Applied across 5 metrics
11    Novelty effect check                              No novelty bias detected
12    Logistic regression + McFadden pseudo R²          R²=0.3325 (good fit)
13    Segment analysis                                  Light users show highest uplift
14    Business impact + ship/no-ship recommendation     DO NOT SHIP
```

---

## 📐 Statistical Methods Reference

| Method | Why I used it | Result |
|--------|--------------|--------|
| Shapiro-Wilk + QQ plot | Check normality before picking a test | NOT normal |
| Mann-Whitney U | Non-parametric — handles skewed, zero-inflated data | p=0.237 ns |
| Two-sample t-test | Secondary comparison against Mann-Whitney | p=0.222 ns |
| Cohen's d | Is the effect practically meaningful? | 0.0034 negligible |
| Bootstrap CI (n=10,000) | CI without normality assumptions | [-0.11%, +0.25%] |
| Power analysis | Was my sample large enough? | Yes — well-powered |
| Bonferroni + BH correction | Account for multiple metrics simultaneously | Applied |
| Novelty effect check | Does effect hold beyond first exposure? | No novelty bias |
| Logistic regression | Which user features predict reorder behavior? | Significant features found |
| McFadden pseudo R² | Goodness-of-fit for logistic model | 0.3325 (good) |
| Winsorization (1st/99th pct) | Reduce outlier influence before regression | Applied |
| SRM check (chi-square) | Validate randomization integrity | PASS |

---

## 💡 What I Would Do Next

The badge doesn't work on the full user base — but that doesn't mean it never works:

1. **Targeted retest on light users only** — this segment showed the highest uplift. A focused test might reach significance where the full-population test didn't.
2. **Test a stronger discount** — KRW 2,000 may not be compelling enough. Try KRW 5,000 and see if the effect size changes.
3. **Test on a higher-visibility surface** — try the cart screen or checkout flow instead of product cards.
4. **CUPED variance reduction** — use each user's pre-experiment reorder history as a covariate to reduce noise without needing more users.

---

## 📂 Folder Structure

```
instacart-ab-test/
│
├── README.md
├── ab_test_instacart_v4.ipynb
├── requirements.txt
└── .gitignore
```

---

## ▶️ How to Run

```bash
# 1. clone
git clone https://github.com/suin8606/instacart-ab-test.git
cd instacart-ab-test

# 2. install dependencies
pip install -r requirements.txt

# 3. set up kaggle credentials
# place kaggle.json in ~/.kaggle/kaggle.json

# 4. run notebook
jupyter notebook ab_test_instacart_v4.ipynb
```

---

## 🗂️ Dataset

[Instacart Market Basket Analysis — Kaggle](https://www.kaggle.com/datasets/psparks/instacart-market-basket-analysis)

| File | Rows | Description |
|------|------|-------------|
| orders.csv | 3,421,083 | Order-level data with user_id, timing, sequence |
| order_products__prior.csv | 32,434,489 | Product-level data with reorder flag |
| products.csv | 49,688 | Product names |

---

## ⚠️ Data Quality Notes

- `days_since_prior_order` is capped at 30 by Instacart — the spike at day 30 is a data artifact, not a real behavioral pattern.
- Zero-inflation is present: a meaningful share of users have reorder_rate = 0 (one-time shoppers). This is why Mann-Whitney was chosen over a t-test.
- Group assignment is simulated via random user ID split on historical data — not a live experiment.

---

## 🏷️ Skills Demonstrated

`Python` `A/B Testing` `Experimental Design` `Statistical Testing`  
`Non-parametric Methods` `Bootstrap Resampling` `Power Analysis`  
`Multiple Testing Correction` `Logistic Regression` `EDA` `Data Visualization`  
`Business Impact Analysis` `kagglehub API`

---

## 👤 Author

**Suin Kim** — M.S. Statistics & Data Science, Baruch College  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/suinkim29)
[![Email](https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:suin.kim29@gmail.com)
