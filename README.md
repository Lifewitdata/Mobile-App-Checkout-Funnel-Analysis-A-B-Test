<div align="center">

# 🛒 A/B Test Analysis — Simplified Checkout

### Did cutting checkout from 5 steps to 2 steps move the needle?

*A full end-to-end product analytics case study — funnel analysis, statistical testing, segment deep-dives, and a ship/hold recommendation.*

<br/>

![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-Data%20Wrangling-150458?style=for-the-badge&logo=pandas&logoColor=white)
![scipy](https://img.shields.io/badge/scipy-Stats-8CAAE6?style=for-the-badge&logo=scipy&logoColor=white)
![Status](https://img.shields.io/badge/Verdict-Ship%20the%20Variant%20✅-2ea44f?style=for-the-badge)

</div>

---

## 🎯 The Question

The Product team believed our 5-step checkout was bleeding conversions — especially on mobile. A simplified 2-step variant collapsed address, delivery, and payment into a single screen.

**Would fewer steps mean more purchases?**

---

## ⚡ TL;DR — Results at a Glance

<div align="center">

| | Control 🔵 | Variant 🟠 | Δ |
|:--|:--:|:--:|:--:|
| 🏆 **Purchase CVR** | 28.9% | 36.3% | **+25.8% lift** ✅ |
| 💰 **Revenue / User** | €50.82 | €65.04 | **+28.0%** |
| 🧾 **Avg Order Value** | €156.24 | €152.48 | -2.4% *(n.s.)* |
| 📦 **Return Rate** | 15.2% | 14.9% | -0.3pp *(n.s.)* |
| ✅ **Checkout Completion** | 60.1% | 80.0% | **+19.9pp** ✅ |

</div>

> 💡 **Guardrail held.** AOV drop is not statistically significant. Return rate unchanged.  
> **Recommendation: Ship globally — except Romania (see below).**

---

## 📂 Repository Structure

```
ab-test-checkout-analysis/
│
├── 📓 ab_test_checkout_analysis.ipynb    ← Main analysis notebook
│
├── 📁 data/
│   ├── users.csv              # 5,000 users  — demographics, device, market
│   ├── orders.csv             # 3,310 orders — value, category, AB group, returns
│   ├── funnel_events.csv      # 49,756 events — 6-stage funnel + time on stage
│   └── ab_test_config.csv     # Test config  — variants, dates, sample targets
│
└── README.md
```

---

## 🔬 Notebook Walkthrough

```
01 · Context & Hypothesis        → test rationale, metrics, guardrail definition
02 · Data Load & Validation      → null checks, balance check, date parsing
03 · Full Funnel Drop-off        → all 6 stages, session-level, delta pp table
04 · Primary Metric — CVR        → z-test, 95% CI, lift significance
05 · Secondary Metrics           → revenue/user, AOV t-test, return rate
06 · Checkout Deep-Dive          → checkout_start → payment_info → order_placed
07 · Segment Analysis            → device · country · age group breakdowns
08 · Romania Anomaly 🚨          → near-zero lift detected, hypotheses raised
09 · Summary & Recommendation    → decision table, ship/hold, next steps
```

---

## 📦 Data Dictionary

<details>
<summary><b>users.csv</b> — 5,000 rows · 6 columns</summary>

| Column | Type | Description |
|:--|:--|:--|
| `user_id` | string | Unique user identifier |
| `country` | string | DE · PL · CZ · SK · RO |
| `device` | string | iOS · Android |
| `user_type` | string | new · returning |
| `registration_date` | date | Account creation date |
| `age_group` | string | 18-24 · 25-34 · 35-44 · 45-54 · 55+ |

</details>

<details>
<summary><b>orders.csv</b> — 3,310 rows · 11 columns</summary>

| Column | Type | Description |
|:--|:--|:--|
| `order_id` | string | Unique order identifier |
| `user_id` | string | FK → users |
| `session_id` | string | FK → funnel_events |
| `order_date` | date | Date of purchase |
| `country` / `device` | string | Market and device |
| `ab_group` | string | control · variant |
| `category` | string | Fashion · Electronics · Sports · Beauty · Home & Garden |
| `order_value_eur` | float | Order value in EUR |
| `items_count` | int | Number of items in order |
| `is_returned` | int | 1 = returned · 0 = kept |

</details>

<details>
<summary><b>funnel_events.csv</b> — 49,756 rows · 9 columns</summary>

| Column | Type | Description |
|:--|:--|:--|
| `event_id` | string | Unique event identifier |
| `session_id` | string | Session grouping key |
| `user_id` | string | FK → users |
| `stage` | string | `app_open` → `product_view` → `add_to_cart` → `checkout_start` → `payment_info` → `order_placed` |
| `event_date` | date | Date of event |
| `device` / `country` | string | Device and market |
| `ab_group` | string | control · variant |
| `time_on_stage_sec` | int | Seconds spent on this stage |

</details>

<details>
<summary><b>ab_test_config.csv</b> — test configuration</summary>

| Column | Description |
|:--|:--|
| `ab_group` | control · variant |
| `variant_name` | Human-readable description |
| `launched_date` / `end_date` | Test window |
| `sample_size_target` | 10,000 users per group |

</details>

---

## ⚙️ Setup

```bash
# 1 · Clone
git clone https://github.com/your-username/ab-test-checkout-analysis.git
cd ab-test-checkout-analysis

# 2 · Install dependencies
pip install pandas numpy scipy matplotlib jupyter

# 3 · Run
jupyter notebook ab_test_checkout_analysis.ipynb
```

> **Note:** CSVs are read from the same directory as the notebook by default. Place all data files alongside the `.ipynb` or update the paths.

---

## 🧮 Statistical Approach

| Decision | Choice | Why |
|:--|:--|:--|
| Primary test | Two-proportion z-test | Binary conversion outcome |
| Significance level | α = 0.05 | Standard product analytics threshold |
| Confidence intervals | 95% Wald | Straightforward, interpretable |
| AOV comparison | Welch independent t-test | Unequal group sizes |
| Unit of analysis | **Unique users** | Avoids session-level inflation |
| Library | `scipy.stats` (no statsmodels) | Zero extra dependencies |

---

## 🚨 Romania — The Anomaly

Romania was the **only market with near-zero lift** (+0.3pp vs ~+9pp everywhere else).

```
                  checkout_start  payment_info  order_placed
Control (RO)          100%           73.3%         58.8%
Variant  (RO)         100%           81.2%         65.1%   ← barely moved
```

Users in Romania reach `payment_info` more often with the variant — but then abandon at a similar rate to control. Leading hypotheses: missing local payment method in the new UI, copy/localisation gaps, or a different payment behaviour profile.

**Status: RO rollout on hold. Investigation sprint recommended.**

---

## 💡 Skills Demonstrated

`A/B test design` · `statistical hypothesis testing` · `funnel analytics` · `segment analysis` · `anomaly detection` · `product decision framing` · `Python` · `pandas` · `scipy` · `matplotlib` · `Jupyter`

---

<div align="center">

*Built as part of a product analytics portfolio.*  
*Feel free to open an issue or reach out with questions.*

</div>
