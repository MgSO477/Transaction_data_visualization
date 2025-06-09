# Customer & Transaction Risk Analytics — Tableau Dashboards

**Dashboards:**

| Topic | Live URL | Direct Embed (copy & paste) |
|-------|----------|-----------------------------|
| Customer Profile | https://public.tableau.com/app/profile/shanmei.liu/viz/Customer_dashboard_17494291925240/1_1?publish=yes | `<iframe src="https://public.tableau.com/views/Customer_dashboard_17494291925240/1_1?:embed=y&:showVizHome=no" width="100%" height="800" frameborder="0"></iframe>` |
| Transaction Overview | https://public.tableau.com/app/profile/shanmei.liu/viz/Transaction_dashboard/1_1?publish=yes | `<iframe src="https://public.tableau.com/views/Transaction_dashboard/1_1?:embed=y&:showVizHome=no" width="100%" height="800" frameborder="0"></iframe>` |

---

## 1  Business Goal

> Provide the **Risk, Fraud-Monitoring and Product teams** with an always-on, self-service portal that answers two questions:
>
> 1. **Who are our customers?**  (geo, demography, credit quality, device security)  
> 2. **How risky are their recent transactions?** (when, where, how much, fraud share, abnormal spend)

---

## 2  Raw Data (Kaggle)

(https://www.kaggle.com/datasets/computingvictor/transactions-fraud-datasets)

Both files were placed in `/data/raw/`.

---

## 3  Cleaning & Feature Engineering

All steps executed in **Python 3.11 + Pandas 2** (see `data_prep.ipynb`):

### 3.1 Customers → `cleaned_customers.csv`

| Step | Code fragment |
|------|---------------|
| Drop 100 %-null columns (`signup_time`, …) | `df.dropna(axis=1, how='all', inplace=True)` |
| Fix types | `dateutil.parser.parse(profile_last_updated)` etc. |
| Geo | kept **latitude / longitude**; stripped extra whitespace from `address` |
| **Age bins** | `df['age_bin'] = (df['age']//5)*5` |
| **Stale flag** ( > 180 days no update) | ```df['stale_flag'] = (today-df['profile_last_updated']).dt.days>180``` |
| Boolean → 0 / 1 | `card_on_dark_web` / `has_chip_card` |
| Final cols | 17 of 22 original |

### 3.2 Transactions → `cleaned_transactions.csv`

| Step | Why |
|------|-----|
| Parse `txn_ts` → `datetime` | later truncate to month |
| Derive `txn_year_month` | `pd.to_datetime(txn_ts).dt.to_period('M')` |
| Remove obvious typos | keep `amount_num` **between -$100 k … +$100 k** |
| Boolean cast | `is_fraud` → `bool` |
| Pre-aggregate for performance | monthly net amount + count (6 M → 72 k rows) |

Aggregations saved to `transactions_monthly.csv`.

---

## 4  Data Model (Tableau)

