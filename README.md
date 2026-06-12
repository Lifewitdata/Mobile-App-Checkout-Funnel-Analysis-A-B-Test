# Mobile-App-Checkout-Funnel-Analysis-A-B-Test
> **Portfolio Project | Data Analyst 
> Tools: Python · Pandas · NumPy · Matplotlib · SciPy  
> Dataset: International Marketplace 

---

## Project overview

operates international online marketplaces where thousands of third-party retailers sell alongside  own products. As the embedded data analyst, I was tasked with identifying why users were dropping off during the mobile app checkout flow and validating whether a simplified checkout design would improve conversion.

This project covers end-to-end funnel analysis, device and country segmentation, A/B test design and statistical significance testing, and post-purchase cohort retention analysis.

---

## Business problem

The product team observed declining checkout completion rates on the mobile app but lacked visibility into **where** users were dropping off and **why**. Key questions:

- Which funnel stage has the highest drop-off rate?
- Does conversion differ by device (Android vs iOS) or country?
- Does a simplified 2-step checkout outperform the existing 5-step flow?
- How does repeat purchase behaviour look in the 6 weeks post-purchase?

---

## Dataset

| File | Rows | Description |
|------|------|-------------|
| `users.csv` | 5,000 | User profiles — country, device, age group, registration date |
| `funnel_events.csv` | 49,756 | Session-level funnel events across 6 stages |
| `orders.csv` | 3,310 | Completed orders with GMV, category, A/B group |
| `ab_test_config.csv` | 2 | A/B test variant definitions |

### Funnel stages
```
app_open → product_view → add_to_cart → checkout_start → payment_info → order_placed
```

### Key columns — funnel_events.csv
| Column | Description |
|--------|-------------|
| `session_id` | Unique session identifier |
| `user_id` | User identifier (joins to users table) |
| `stage` | Funnel stage name |
| `ab_group` | `control` = 5-step checkout / `variant` = 2-step checkout |
| `device` | Android or iOS |
| `country` | DE, PL, CZ, SK, RO |

---

## Project steps

### Step 1 — Load & inspect datasets
```python
users   = pd.read_csv('users.csv', parse_dates=['registration_date'])
funnel  = pd.read_csv('funnel_events.csv', parse_dates=['event_date'])
orders  = pd.read_csv('orders.csv', parse_dates=['order_date'])
```

### Step 2 — Overall funnel drop-off analysis
```python
STAGES = ['app_open','product_view','add_to_cart','checkout_start','payment_info','order_placed']
stage_sessions = funnel.groupby('stage')['session_id'].nunique().reindex(STAGES)
funnel_df['cvr_from_top']  = stage_sessions / stage_sessions.iloc[0] * 100
funnel_df['step_drop_pct'] = funnel_df['sessions'].pct_change().abs() * 100
```

### Step 3 — Segment by device & country
```python
def conversion_rate(df, group_col):
    top  = df[df['stage'] == 'app_open'].groupby(group_col)['session_id'].nunique()
    conv = df[df['stage'] == 'order_placed'].groupby(group_col)['session_id'].nunique()
    result = pd.DataFrame({'sessions': top, 'orders': conv}).fillna(0)
    result['cvr_%'] = (result['orders'] / result['sessions'] * 100).round(2)
    return result
```

### Step 4 — A/B test statistical significance (Chi-square)
```python
from scipy import stats
contingency = np.array([[ctrl_converted, ctrl_dropped],
                        [var_converted,  var_dropped]])
chi2, p_value, dof, _ = stats.chi2_contingency(contingency)
lift = (var_cvr - ctrl_cvr) / ctrl_cvr * 100
```

### Step 5 — 6-week post-purchase cohort retention
```python
first_order = orders.groupby('user_id')['order_date'].min().reset_index()
orders_m = orders.merge(first_order, on='user_id')
orders_m['weeks_since_first'] = (
    (orders_m['order_date'] - orders_m['first_order_date']).dt.days // 7
)
cohort = orders_m.groupby('weeks_since_first')['user_id'].nunique()
```

### Step 6 — Key findings & recommendations

---

## Results

| Metric | Value |
|--------|-------|
| Biggest funnel drop-off | `add_to_cart` — 40.6% of sessions lost |
| Overall app conversion rate | 19.0% (app open → order placed) |
| iOS CVR | 20.1% |
| Android CVR | 18.2% |
| Control checkout CVR | 60.1% |
| Variant checkout CVR | 80.0% |
| A/B test lift | **+33.2%** |
| Statistical significance | p < 0.0001 ✓ |
| Best performing country | DE — 23.6% CVR |
| Weakest performing country | SK — 11.1% CVR |
| Reporting time saved | ~90% via automation |

---

## Key recommendations

1. **Roll out simplified 2-step checkout** to 100% of users — statistically significant +33.2% lift
2. **Investigate Android-specific bugs** — 1.9pp gap vs iOS at scale affects thousands of sessions
3. **Localise checkout experience for SK & RO** — converting at half the rate of DE
4. **Launch post-purchase email campaign** — repeat purchase rate drops sharply after Week 1

---

## How to run

```bash
# Install dependencies
pip install pandas numpy matplotlib scipy

# Run analysis
python3 kaufland_project_analysis.py
```

Place all 4 CSV files in the same directory as the script before running.

---

## Skills demonstrated

`Python` `Pandas` `NumPy` `Matplotlib` `SciPy` `A/B Testing` `Chi-square test` `Funnel Analysis` `Cohort Analysis` `Mobile App Analytics` `Statistical Significance` `Data Visualisation` `Stakeholder Reporting`

---

*Project built as part of Data Analyst portfolio — Weblays Technologies client work (Kaufland International)*
