# KPI Report — Digital Payment Analytics (UPI Transactions 2024)

**Task:** AI & ML Internship – Task 13: Digital Payment Analytics
**Prepared by:** Data Analyst (Intern)
**Dataset:** upi_transactions_2024.csv (5,020 raw records → 5,000 records after cleaning)

---

## 1. Why these KPIs?

Before jumping into numbers, I first thought about *who* would actually read this dashboard. Leadership doesn't care about every column in the CSV — they care about two things: **is the payment system healthy**, and **where is money and risk concentrated**. So I split my KPIs into two buckets:

- **Payment Health KPIs** → tell the Operations/Risk team if the system is working properly (success rate, failure rate, peak load).
- **Business KPIs** → tell Product/Business team where revenue and users are coming from (volume, revenue by method, active users).

This split directly answers the two questions the task asked: *"Which KPIs define payment health?"* and *"Which KPIs help operations teams?"*

---

## 2. Data Cleaning Summary (Phase 1 – before KPIs were calculated)

| Step | What I found | What I did |
|---|---|---|
| Missing `amount` | 50 rows missing | Filled with **median** amount (median is safer than mean because transaction amounts are skewed by a few very large payments) |
| Missing `device_type` | 30 rows missing | Filled with the **mode** (most frequent device, Android) |
| Duplicate transactions | 20 duplicate `transaction_id` rows | Dropped duplicates, keeping first occurrence |
| `timestamp` | Was stored as text | Converted to datetime and derived `hour`, `day_name`, `month_name` |

**Final clean dataset: 5,000 transactions, 500 unique users, 0 missing values.**

Derived columns created:
- `peak_hour_flag` → 1 if transaction hour is in {10-14, 18-21} (lunch + evening rush)
- `high_value_flag` → 1 if amount is in the top 10% (≥ ₹2,990.30)
- `user_failure_rate` → per-user % of failed transactions

---

## 3. Mandatory KPIs — Actual Values

| # | KPI | Value | My Interpretation |
|---|---|---|---|
| 1 | **Total Transactions** | 5,000 | Total volume of activity processed in the dataset period |
| 2 | **Total Payment Volume** | ₹1,06,99,470.62 (~₹1.07 Cr) | Total money moved across all methods |
| 3 | **Success Rate** | 87.94% | Roughly 9 in 10 transactions complete fine |
| 4 | **Failure Rate** | 8.74% | ~1 in 11 transactions fails — this is the number Risk team should watch |
| 5 | **(Pending Rate — added by me)** | 3.32% | Not asked in the task, but I added it because "stuck" transactions are a real customer-experience problem too |
| 6 | **Average Transaction Value** | ₹2,139.89 | Typical spend per transaction |
| 7 | **Peak Transaction Hour** | 12:00 (noon) | Most transactions happen around lunchtime |
| 8 | **Active Users** | 500 | Every user in the dataset transacted at least once |

> **Note on Failure Rate:** the task brief mentions complaints about failures went up 22%. My data doesn't have a "before" period to compare against, so I can't verify that 22% figure directly — but an 8.74% baseline failure rate is already high enough that even small spikes would be noticeable to customers, which supports the business scenario.

---

## 4. Revenue Contribution by Payment Method

| Payment Method | Txn Count | Total Volume (₹) | % of Total Volume | Success Rate |
|---|---|---|---|---|
| UPI | 2,410 | 49,05,703 | 45.85% | 87.18% |
| Card | 1,029 | 24,16,752 | 22.59% | 88.73% |
| Wallet | 759 | 18,06,303 | 16.88% | 87.35% |
| NetBanking | 528 | 10,93,794 | 10.22% | **90.72% (best)** |
| QR | 274 | 4,76,919 | 4.46% | 87.96% |

**My reading of this table:** UPI dominates in both count and volume — nearly half of all money flows through it, which matches the real-world UPI boom in India. Interestingly, NetBanking has the *fewest* users but the *best* success rate, while UPI and Wallet (the most-used methods) have slightly below-average reliability. That's a useful insight for the risk team: the most-used rail is not the most reliable one.

---

## 5. Failure Pattern (Time vs Failure Rate)

Failure rate by hour (highest first):

| Hour | Failure Rate |
|---|---|
| 09:00 | 12.87% |
| 13:00 | 11.65% |
| 08:00 | 10.86% |
| 00:00 (midnight) | 10.80% |
| 04:00 | 10.55% |

**Insight:** Failures spike around 9 AM and 1 PM — both are transition points into high-traffic windows (morning start-of-day and post-lunch). Midnight/early-morning hours (12 AM–4 AM) also show elevated failure rates, which could point to backend maintenance windows or batch jobs competing for system resources during off-peak hours. This is exactly the kind of "operational problem" pattern the task wanted us to look for.

---

## 6. City / Merchant Category Split

**City-wise transaction count** (fairly balanced, no single city dominates):
Hyderabad (687) > Pune (646) > Ahmedabad (632) > Kolkata (623) > Chennai (616) > Bangalore (605) > Mumbai (597) > Delhi (594)

**Merchant Category by Revenue:**
Fuel (₹14.15L) > Healthcare (₹13.84L) > Shopping (₹12.42L) > Transport (₹11.87L) > Entertainment (₹11.37L) > Grocery (₹11.26L) > Food (₹10.90L) > Utilities (₹10.88L) > Education (₹10.31L)

Revenue is spread fairly evenly across categories (no category is more than ~14% of total), so this looks like a generic day-to-day spending dataset rather than one dominated by a single vertical like e-commerce.

---

## 7. User Segmentation KPIs

I segmented all 500 users using their transaction count and total spend (75th/25th percentile cutoffs), since the task didn't give exact segment rules and asked us to design our own.

| Segment | Users | % of Users | Total Revenue (₹) | % of Revenue | Avg Txns/User | Avg Failure Rate |
|---|---|---|---|---|---|---|
| Heavy + High-Value User | 64 | 12.8% | 28,47,878 | 26.62% | 14.08 | — |
| High-Value User | 61 | 12.2% | 24,48,924 | 22.89% | 9.18 | — |
| Heavy User | 98 | 19.6% | 18,21,351 | 17.02% | 13.29 | — |
| Regular/Frequent Transactor | 125 | 25.0% | 18,04,445 | 16.86% | 9.86 | — |
| Low-Engagement User | 152 | 30.4% | 17,76,873 | 16.61% | 6.61 | — |

**This is the single most important business insight in the whole project:** the top two segments — **Heavy + High-Value** and **High-Value** users — are only **25% of the user base but generate ~49.5% of total revenue.** Classic 80/20-style concentration. Meanwhile, **Low-Engagement users are the single largest group by headcount (30.4%)** but contribute the least revenue per user. This tells Product team exactly where retention effort vs. reward campaigns should be targeted.

---

## 8. Answering the Business Questions

1. **Which payment methods perform best?** UPI leads on volume/count, but NetBanking is the most *reliable* (90.72% success). "Best" depends on whether you mean scale or reliability.
2. **Which time periods experience failures?** Early morning (12–4 AM) and two daily spikes at 9 AM and 1 PM.
3. **Which users contribute most revenue?** The top 25% of users (Heavy/High-Value segments) drive ~half of all revenue.
4. **Which regions need attention?** No city is a major outlier, but Delhi and Mumbai have the lowest transaction counts — worth checking if that's lower adoption or a local service issue.
5. **What should product/ops teams do?** See Recommendations in the README.

---

## 9. Honest Limitations (things I'd flag before presenting this to leadership)

- The dataset doesn't include a "reason for failure" column, so I can only say *when* failures spike, not *why*.
- No pre-2024 data exists, so I could not verify the "22% increase in complaints" mentioned in the business scenario.
- Segmentation thresholds (75th/25th percentile) are one reasonable choice, not the only valid one — a business team might define "high value" differently.
