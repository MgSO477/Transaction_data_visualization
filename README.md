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

https://www.kaggle.com/datasets/computingvictor/transactions-fraud-datasets

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

## 4  Dashboards   

---

### 5.1  Customer-Profile & Risk Dashboard  

| 📊 Chart | How it was built (1-line logic) | What you should read out loud (business insight) |
|----------|---------------------------------|--------------------------------------------------|
| **Customer Age Distribution** – stacked bars | Age binned every 5 years, split by gender | “Half of our 2 000 customers are 25-45; females lead below 30, males above 40.” |
| **Yearly Income Statistics** – horizontal bars | Income split into \$8 k steps, again coloured by gender | “Income skews male in the \$24-40 k bracket; female share is bigger in the entry segment.” |
| **Credit-Score Category** – simple bar | Counts per FICO band | “Good (700-799) is our core – 45 % of the book; only 3 % sit in ‘Poor’.” |
| **Yearly Income through Ages** – box-and-whisker | One box per 5-year age bin | “Median income climbs till late 40 s, plateaus, then drops. Outliers above \$250 k are all < 50 years old.” |
| **Credit-Score Heat Map** (USA) | Map, colour = score category | “High-score clusters sit on the Pacific Coast and north-east; centre-south still trending ‘Fair’.” |
| **Debt-to-Income Heat Map** (USA) | Map, colour = DTI% | “High DTI hot-spots light up the Rust Belt and Florida – targets for refinancing-offers.” |

*No global filters were added on purpose – the page is meant for a first-look conversation; maps and box-plots are still clickable if you want to isolate a region or an age group.*

---

### 5.2  Transaction-Risk Dashboard  

| 📊 Chart | Build logic | What it tells the business |
|----------|------------|-----------------------------|
| **Industry Fraud Rate** – blue bars | Fraud-count ÷ total-count for each MCC | “Grocery, Misc. Food Stores and Service Stations are the top three fraud-dense sectors.” |
| **Transaction Amount by Industry** – orange bars, colour = fraud flag | Sum(amount) per MCC; bars re-coloured red as fraud share rises | “The ‘NULL’ merchant code (data-quality gap) hides \$60 M in sales – and most of the fraud. Clean that field.” |
| **Fraud Count through Cities** – bar chart | Top-N cities by fraud count | “Houston, Miami and Brooklyn account for > 15 % of all fraudulent hits.” |
| **Fraud-Rate Timeline** – blue line | Monthly fraud-rate (2010-2017) | “Fraud waves arrived in Q3-2013 and late-2015 – worth checking rule-changes or data-leaks back then.” |
| **Monthly Amount vs Txn Count** – dual-axis | Grey bars = counts, orange line = net spend | “Spend gradually climbs, but ticket-size stays flat – growth is volume-driven, not higher baskets.” |
| **Fraud Heat Map** (all US dots) | Every dot a transaction; colour depth = fraud count | “South-west & Florida show the darkest reds; supports the city-ranking above.” |

**How the page is meant to be used**  
1. Start with the *Industry Fraud Rate* bar – hover a bar, Tableau highlights the same MCC in the “Amount” chart on the right.  
2. Click any city in the *Fraud through Cities* view – all other charts instantly redraw, so the manager can ask *“what does Houston spend look like?”* without extra filters.  
3. Use the *timeline* to see if a spike is a one-off or a trend; forward the date and MCC to the fraud-rule owner.

---

### 5.3 Steps 

1. **Raw data** – two Kaggle CSVs (customer & transaction).  
2. **Clean-up** – dropped empty columns, fixed dates, capped impossible amounts (> \$1 M), added 5-year age bins & DTI buckets.  
3. **Join** – left-joined on `client_id`; keeps customers even if they made zero recent purchases.  
4. **Aggregations** – pre-aggregated transactions to monthly level to keep the Tableau file under 25 MB.  
5. **Design choices**  
   - Customers page = *demography first*, no filters, easy ice-breaker.  
   - Transactions page = *risk first*, each bar / dot acts as a filter, encourages exploration.  
6. **Publish & share** – pushed to Tableau Public, embedded via `<iframe>` so anyone (even without Tableau) can play with the numbers.
 
> *“These two pages let you see who our customers are, where the money flows, and where fraud hides. ”*

