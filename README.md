# Instacart A/B Test: Does a Discount Badge Increase Reorder Rates?

**Can a discount badge drive more reorders — and is the effect real enough to ship?**  
A full end-to-end A/B testing pipeline using Instacart's public dataset (3.4M orders, 200K users).

---

## The Business Question

Food delivery and grocery platforms live and die by **retention**. Getting a user to reorder is significantly cheaper than acquiring a new one. I simulated a real product experiment:

> *If we show users a discount badge on product cards, does it meaningfully increase reorder rates — and does the revenue justify the discount cost?*

This mirrors the type of experimentation run daily at companies like Coupang Eats, DoorDash, and Instacart.

---

## Result: Do Not Ship

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
| Final decision | **Do Not Ship** |

**Why not ship despite a positive net gain?**

The p-value of 0.237 is well above the 0.05 threshold — meaning the observed +0.17% uplift has a 24% chance of being pure random noise. The 95% bootstrap CI crosses zero (-0.110% to +0.251%), which means the true effect could easily be negative.

Importantly, this is NOT a sample size problem. My 103,104 users per group far exceeds the 5,566 required to detect a 1pp effect. The test is well-powered — the non-significance is a genuine finding.

Shipping on this result would be a coin flip. I'd recommend retesting with a targeted strategy instead (details in "What I Would Do Next" below).

---

## Key Charts

### When Do Users Order?
> Peak ordering is around 10am–3pm. Sunday and Monday are the busiest days — useful context for deciding when to run future experiments.

---

### Reorder Rate Distribution: Control vs Variant
> Both groups show nearly identical distributions — visually consistent with a non-significant result.



---

### Normality Check — QQ Plots
> Both groups deviate clearly from normality. The left tail sits flat at y=0 — this is zero-inflation from one-time shoppers with reorder_rate = 0. This is exactly why I chose Mann-Whitney U over a t-test.



---

### Bootstrap Confidence Interval (10,000 resamples)
> The CI crosses zero, confirming we cannot be confident the variant is genuinely better than control.



---

### Power Analysis
> My sample of 103,104 per group is 18x larger than what was needed. Non-significance here means the effect is genuinely small — not that the test was underpowered.



---

### Novelty Effect Check
> Early uplift (+0.13%) and late uplift (+0.08%) are both tiny and consistent — no novelty bias detected. The badge simply doesn't move the needle regardless of exposure timing.



---

### Segment Analysis
> Light users (1-5 orders) showed the highest uplift. If I retested, I'd target only this segment rather than the full user base.



---

### Business Impact
> Net gain looks positive (KRW 14M/month) but is built on a statistically insignificant uplift. This number cannot be trusted — it could just as easily be negative.



---

## How to Export Charts From the Notebook

After each `plt.show()` call in the notebook, add a save line:

```python
plt.savefig('img/01_orders_timing.png', dpi=150, bbox_inches='tight')
plt.show()
```

Then commit the `img/` folder and the images will render in this README automatically.

---

## My Full Analysis Pipeline

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

## Statistical Methods Reference

| Method | Why I used it | Result |
|--------|--------------|--------|
| Shapiro-Wilk + QQ plot | Check normality before picking a test | NOT normal |
| Mann-Whitney U | Non-parametric — handles skewed, zero-inflated data | p=0.237 ns |
| Two-sample t-test | Secondary comparison against Mann-Whitney | p=0.222 ns |
| Cohen's d | Is the effect practically meaningful, not just statistically real? | 0.0034 negligible |
| Bootstrap CI (n=10,000) | CI without normality assumptions | [-0.11%, +0.25%] |
| Power analysis | Was my sample large enough to detect a real effect? | Yes — well-powered |
| Bonferroni + BH correction | Account for testing multiple metrics simultaneously | Applied |
| Novelty effect check | Does effect hold beyond first exposure? | No novelty bias |
| Logistic regression | Which user features predict reorder behavior? | Significant features found |
| McFadden pseudo R² | Goodness-of-fit for logistic model (R² equivalent) | 0.3325 (good) |
| Winsorization (1st/99th pct) | Reduce outlier influence before regression | Applied |
| SRM check (chi-square) | Validate randomization integrity before trusting results | PASS |

---

## What I Would Do Next

The badge doesn't work on the full user base — but that doesn't mean it never works:

1. **Targeted retest on light users only** — this segment showed the highest uplift. A focused test might reach significance where the full-population test didn't.
2. **Test a stronger discount** — KRW 2,000 may not be compelling enough. Try KRW 5,000 and see if the effect size changes.
3. **Test on a higher-visibility surface** — the badge on product cards may not be noticed. Try the cart screen or checkout flow instead.
4. **CUPED variance reduction** — use each user's pre-experiment reorder history as a covariate to reduce noise and increase test sensitivity without needing more users.

---

## Data Quality Notes

- `days_since_prior_order` is capped at 30 by Instacart. The spike at day 30 in the distribution chart is a data artifact — it includes all gaps ≥ 30 days, not a real behavioral pattern.
- Zero-inflation is present: a meaningful share of users have reorder_rate = 0 (one-time shoppers). This is why Mann-Whitney was chosen over a t-test.
- Group assignment is simulated via random user ID split on historical data — not a live experiment.

---

## Folder Structure

```
instacart-ab-test/
│
├── README.md
├── ab_test_instacart_v4.ipynb
├── requirements.txt
├── .gitignore
└── img/
    ├── 01_orders_timing.png
    ├── 02_reorder_distribution.png
    ├── 03_qq_plots.png
    ├── 04_bootstrap_ci.png
    ├── 05_power_analysis.png
    ├── 06_novelty_effect.png
    ├── 07_segment_analysis.png
    └── 08_business_impact.png
```

---

## How to Run

```bash
# 1. clone
git clone https://github.com/your-username/instacart-ab-test.git
cd instacart-ab-test

# 2. install dependencies
pip install -r requirements.txt

# 3. set up kaggle credentials
# place kaggle.json in ~/.kaggle/kaggle.json

# 4. create img folder
mkdir img

# 5. run notebook
jupyter notebook ab_test_instacart_v4.ipynb
```

---

## Dataset

[Instacart Market Basket Analysis — Kaggle](https://www.kaggle.com/datasets/psparks/instacart-market-basket-analysis)

| File | Rows | Description |
|------|------|-------------|
| orders.csv | 3,421,083 | Order-level data with user_id, timing, sequence |
| order_products__prior.csv | 32,434,489 | Product-level data with reorder flag |
| products.csv | 49,688 | Product names |

---

## Skills Demonstrated

`Python` `A/B Testing` `Experimental Design` `Statistical Testing`  
`Non-parametric Methods` `Bootstrap Resampling` `Power Analysis`  
`Multiple Testing Correction` `Logistic Regression` `EDA` `Data Visualization`  
`Business Impact Analysis` `kagglehub API`

---

## Author

**Suin Kim**  
M.S. Statistics & Data Science — Baruch College  
[linkedin.com/in/suinkim29](https://linkedin.com/in/suinkim29) · suin.kim29@gmail.com
