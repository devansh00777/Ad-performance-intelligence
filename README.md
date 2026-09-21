# 📣 Ad Performance Analytics

**An end-to-end digital advertising analytics project** — 400,000 ad events across 50 campaigns, 200 ads, and 2 platforms, analyzed from raw event logs through to a three-page Power BI dashboard. Built entirely in Python (pandas) with DAX-driven reporting on top.

📊 **Dataset:** 400,000 events · 50 campaigns · 200 ads · 10,000 users · 2 platforms · 4 ad formats &nbsp;|&nbsp; 🛠️ **Stack:** Python (pandas, seaborn) · Power BI (DAX)

📅 **Observation window:** May 7, 2025 – Aug 6, 2025 (91 days)

---

## 💡 What This Does

This project turns raw ad-event logs (impressions, clicks, purchases, likes, comments, shares) into an answer to the question that actually matters: **what turns exposure into revenue, and where does that break down?** Not "how many events happened" — which campaigns, platforms, and ad formats convert, and whether campaign size has anything to do with campaign quality.

Every number below was checked against the cleaned source data before being written down — including two places where an earlier version of the analysis was wrong, and the fix is documented in the notebooks themselves.

## 🔄 Project Pipeline

1. **Raw layer** — `events`, `ads`, `campaigns`, `users` loaded 1:1 from CSV, no transformation.
2. **Clean layer** (`01_data_cleaning_combined.ipynb`) — primary/foreign-key validation, business-rule checks, derived-field verification (`day_of_week`, `time_of_day` recomputed and diffed against the source, not trusted blindly), and an explicit decision to exclude `users` from the core model rather than paper over a real identifier problem.
3. **Analysis layer** (`02_eda_and_performance.ipynb`) — exploratory analysis, KPI calculation, and funnel analysis combined into one notebook, built around impressions → clicks → purchases rather than raw event counts.
4. **Dashboard** — Power BI report built directly on the cleaned tables.

## 🗂️ Dashboard Sections

| Page | Visuals |
|---|---|
| 📈 **Executive Overview** | KPI cards (Impressions, Clicks, Purchases, CTR, Click-to-Purchase Rate, Purchase Rate) · Advertising Event Funnel · Event Volume by Type · Daily Event Volume trend |
| 🎯 **Campaign Performance** | Campaign Impressions vs. Purchases (scatter) · 10 Highest Campaign Purchase Rates · Full Campaign Performance table |
| 📡 **Channel & Ad Type Performance** | Impressions by Platform (donut) · Platform KPI Comparison · Ad Type Performance table · Impressions by Ad Type (donut) |

## 🗄️ Data Model

```
CORE TABLES
├─ events_clean      → one row per ad event (Impression / Click / Purchase / Like / Comment / Share)
├─ ads_clean          → one row per ad; links to campaign, carries platform + ad type
└─ campaigns_clean    → one row per campaign

RELATIONSHIP
campaigns_clean (1) ──< ads_clean (many) ──< events_clean (many)

EXCLUDED FROM CORE MODEL
└─ users_clean        → exported for reference only. 50 of 10,000 user_ids repeat with
                         genuinely conflicting attributes (age, gender, country, etc.),
                         so user_id can't be trusted as a join key. Not merged into
                         events; not used in any dashboard visual.
```

## ❓ Business Questions & Analysis

| # | Question | Where It's Answered |
|---|---|---|
| 1 | Is the source data clean enough to build on? | `01_data_cleaning_combined.ipynb` |
| 2 | What does the event data actually look like, and where is the class imbalance? | `02_eda_and_performance.ipynb` — Part A |
| 3 | Which platforms, ad types, and campaigns convert impressions into purchases? | `02_eda_and_performance.ipynb` — Part B, *Channel & Ad Type* and *Campaign Performance* pages |
| 4 | Where does the funnel lose the most volume? | `02_eda_and_performance.ipynb` — Part C, *Executive Overview* page |
| 5 | Does campaign size (more ads, more spend) actually mean better conversion? | `02_eda_and_performance.ipynb` — Section A6 |

## 🧰 Techniques Used

- Primary-key and foreign-key validation with hard `assert`s, not just printed counts
- Derived-field verification — `day_of_week` and `time_of_day` recomputed from `timestamp` and diffed against the supplied columns rather than trusted
- Multi-table joins with `validate="many_to_one"` to guarantee referential integrity on every merge
- Pivot tables / crosstabs for KPI aggregation by platform, ad type, and campaign
- Correlation checks used specifically to catch tautological findings — e.g. confirming that "bigger campaigns get more purchases" is a volume effect, not a quality signal, before reporting it as one
- DAX measures in Power BI for CTR, Click-to-Purchase Rate, and Purchase Rate, calculated live rather than stored as static columns

## 📐 Key Metrics

| Metric | Formula | What It Tells You |
|---|---|---|
| CTR | Clicks / Impressions × 100 | How often exposure leads to engagement |
| Click-to-Purchase Rate | Purchases / Clicks × 100 | How often engagement converts to revenue |
| Purchase Rate | Purchases / Impressions × 100 | End-to-end conversion efficiency |

## 🔍 Key Insights

**1. 🏆 One campaign is a genuine standout, not a statistical fluke.**
Campaign_27_Q3 leads on both click-to-purchase rate (8.40% vs. a 2.44%–8.40% range) and purchase rate (0.99%), converting 33 purchases from 3,323 impressions and 393 clicks. Campaign_36_Q3, with nearly identical volume (3,424 impressions, 409 clicks), converts only 10 purchases. Same exposure, 3x the outcome — worth a creative/targeting teardown to find what's replicable.

**2. 📦 Campaign size drives volume, not conversion quality.**
Impressions and purchases correlate strongly at the campaign level (r = 0.93) — but that's because campaigns with more ads generate more of everything. Number of ads vs. purchase rate correlates at essentially zero (r = -0.016). A campaign with 8 ads isn't a better campaign than one with 2; it's just a bigger one.

**3. 📡 Platform and ad format differences are real, but small — and shouldn't be the main lever.**
Facebook edges Instagram on purchase rate (0.61% vs. 0.57%); Stories edges out Image as the best-converting format (0.63% vs. 0.55%). Both real, both under half a percentage point. Platform and ad-type strategy matters far less than which specific campaign you're running.

**4. 📅 Campaign dates don't reflect when events actually happened.**
56.35% of events fall outside their campaign's stated start/end window — more than half the dataset. Campaign dates are kept as metadata but were never used to filter or bucket events for that reason; this points to a tracking/instrumentation gap upstream, not a data-cleaning fix.

**5. 🪪 User identity isn't reliable enough to build a user-level view yet.**
50 of 10,000 `user_id`s repeat with real, conflicting attributes (different age, gender, country on the same ID). Rather than merge a broken key into the core model, `users_clean` is exported separately and excluded from every downstream table and every dashboard visual.

## ✅ Recommendations

- 🏆 Run a creative and audience-targeting teardown on Campaign_27_Q3 specifically — it's outperforming on conversion at comparable volume to its peers, which makes it a genuine template rather than a lucky outlier.
- 📦 Stop using ad count or spend as a proxy for campaign quality in planning conversations — pair every volume metric with a conversion metric before calling a campaign "successful."
- 📡 Don't over-invest in platform or ad-format optimization as a primary strategy — the spread there is real but small; campaign-level execution is the bigger lever by a wide margin.
- 📅 Fix campaign date tracking at the source before using campaign start/end dates for any time-bounded reporting or attribution — right now they can't be trusted for that.
- 🪪 Enforce a real uniqueness constraint on `user_id` upstream (at collection, not at cleaning) before attempting any user-level personalization, retention, or journey analysis.

## 📁 Folder Structure

```
Ad-Performance-Analytics/
│
├───notebooks/
│       01_data_cleaning_combined.ipynb
│       02_eda_and_performance.ipynb
│
├───data/
│       events_clean.csv
│       ads_clean.csv
│       campaigns_clean.csv
│       users_clean.csv
│
├───dashboard/
│       main.pbix
│
└───README.md
```

## 🛠️ Tech Stack

| Category | Tools |
|---|---|
| Data Cleaning & Analysis | Python — pandas, numpy |
| Visualization | matplotlib, seaborn |
| BI / Dashboard | Power BI (DAX) |
| Version Control | Git, GitHub |
