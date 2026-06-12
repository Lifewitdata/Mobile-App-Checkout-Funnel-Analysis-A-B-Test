# 🛒 A/B Test Analysis — Simplified Checkout

> **Did reducing checkout from 5 steps to 2 steps improve purchase conversion?**  
> A full end-to-end product analytics case study using real funnel, order, and user data.

---

## 📌 Overview

This project contains a complete A/B test analysis conducted for a fast-growing mobile e-commerce app. The Product team hypothesised that a 5-step checkout flow was creating unnecessary friction — especially on mobile. A simplified 2-step variant was tested against the standard flow across five Central & Eastern European markets.

**Markets:** DE, PL, CZ, SK, RO  
**Sample size:** ~4,400 users per group  
**Primary metric:** Purchase Conversion Rate (CVR)

---

## 📊 Key Results

| Metric | Control (5-step) | Variant (2-step) | Change | Significant? |
|---|---|---|---|---|
| Purchase CVR | 28.9% | 36.3% | **+25.8% lift** | ✅ p < 0.001 |
| Revenue per User | €50.82 | €65.04 | **+28.0%** | — |
| Average Order Value | €156.24 | €152.48 | -2.4% | ✗ n.s. |
| Return Rate | 15.2% | 14.9% | -0.3pp | ✗ n.s. |
| Checkout Completion | 60.1% | 80.0% | **+19.9pp** | ✅ p < 0.001 |

**Verdict: Ship the variant.** The guardrail (AOV) held. Return rate was unchanged. Revenue per user up +28%.

---

## 📁 Repository Structure

```
├── ab_test_checkout_analysis.ipynb   # Main analysis notebook
├── data/
│   ├── users.csv                     # 5,000 users — country, device, age group, user type
│   ├── orders.csv                    # 3,310 orders — value, category, returns, AB group
│   ├── funnel_events.csv             # 49,756 events — 6-stage funnel with time on stage
│   └── ab_test_config.csv            # Test config — variant names, dates, target sample
└── README.md
```

---

## 🔬 Analysis Structure

The notebook is organised into 9 sections:

1. **Context & Hypothesis** — test rationale, metrics definition, guardrail setup
2. **Data Load & Validation** — null checks, date parsing, group balance sanity check
3. **Full Funnel Walkthrough** — all 6 stages, session-based drop-off table + charts
4. **Primary Metric — CVR** — user-level conversion rate, z-test for proportions, 95% CI
5. **Secondary Metrics** — revenue per user, AOV t-test, return rate
6. **Checkout Deep-Dive** — checkout_start → payment_info → order_placed completion rates
7. **Segment Analysis** — CVR breakdown by device, country, and age group
8. **Romania Flag** — anomaly detection: near-zero lift in RO, root cause hypotheses
9. **Summary & Recommendation** — decision table, ship/hold rationale, next steps

---

## 📦 Data Dictionary

### `users.csv`
| Column | Type | Description |
|---|---|---|
| user_id | string | Unique user identifier |
| country | string | DE, PL, CZ, SK, RO |
| device | string | iOS or Android |
| user_type | string | new or returning |
| registration_date | date | Account creation date |
| age_group | string | 18-24, 25-34, 35-44, 45-54, 55+ |

### `orders.csv`
| Column | Type | Description |
|---|---|---|
| order_id | string | Unique order identifier |
| user_id | string | FK → users |
| session_id | string | FK → funnel_events |
| order_date | date | Date of purchase |
| country / device | string | Market and device |
| ab_group | string | control or variant |
| category | string | Fashion, Electronics, Sports, Beauty, Home & Garden |
| order_value_eur | float | Order value in EUR |
| items_count | int | Number of items |
| is_returned | int | 1 = returned, 0 = kept |

### `funnel_events.csv`
| Column | Type | Description |
|---|---|---|
| event_id | string | Unique event identifier |
| session_id | string | Session grouping key |
| user_id | string | FK → users |
| stage | string | app_open → product_view → add_to_cart → checkout_start → payment_info → order_placed |
| event_date | date | Date of event |
| device / country | string | Market and device |
| ab_group | string | control or variant |
| time_on_stage_sec | int | Seconds spent on this stage |

### `ab_test_config.csv`
| Column | Description |
|---|---|
| ab_group | control or variant |
| variant_name | Human-readable description |
| launched_date / end_date | Test window |
| sample_size_target | 10,000 per group |

---

## ⚙️ Setup & Usage

### Requirements

```bash
Python 3.8+
pandas
numpy
scipy
matplotlib
```

### Run the notebook

```bash
# Clone the repo
git clone https://github.com/your-username/ab-test-checkout-analysis.git
cd ab-test-checkout-analysis

# Install dependencies
pip install pandas numpy scipy matplotlib jupyter

# Launch notebook
jupyter notebook ab_test_checkout_analysis.ipynb
```

> **Note:** The notebook reads CSV files from the same directory by default (`pd.read_csv('users.csv')`). Place the data files in the same folder as the notebook, or update the paths to match your local setup.

---

## 🧠 Statistical Methodology

- **Test type:** Two-proportion z-test for CVR (primary metric)
- **Significance level:** α = 0.05
- **Confidence intervals:** 95% (Wald method)
- **AOV comparison:** Independent samples t-test (Welch)
- **Return rate:** Two-proportion z-test
- **Unit of analysis:** Unique users (not sessions), to avoid session-level inflation
- **No statsmodels dependency** — all tests implemented with `scipy.stats` + manual proportion z-test

---

## 🚨 Notable Finding — Romania

Romania was the only market with near-zero lift (+0.3pp vs +9pp average elsewhere). The checkout sub-funnel shows the variant improved payment_info reach (73% → 81%) but the final order placement rate barely moved (58.8% → 65.1%). Hypotheses include a payment method mismatch, localisation gaps on the new screen, or a different customer profile. **Rollout to RO is on hold pending investigation.**

---

## 💡 Skills Demonstrated

- A/B test design & statistical analysis
- Funnel analytics & drop-off modelling
- Segment analysis (device, geo, age cohort)
- Anomaly detection & root cause framing
- Product decision-making from data
- Python (pandas, scipy, matplotlib)
- Jupyter notebook storytelling

---

## 📬 Contact

Built as part of a product analytics portfolio.  
Feel free to open an issue or reach out with questions.
