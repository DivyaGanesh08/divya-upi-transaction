# Project Report — Digital Payment Analytics (UPI Transactions)

**My Name:** [Your Name]
**Task:** Task 13 – Digital Payment Analytics (AI & ML Internship Program)
**Dataset used:** upi_transactions_2024.csv (5,020 rows, 500 users)
**Tools used:** Python (pandas, matplotlib, seaborn) + Power BI

---

## 1. What was the problem, in simple terms?

Basically, I was told to imagine I'm working as a Data Analyst at a digital payments company (like a PhonePe/GPay type company). The company noticed that customer complaints about failed/delayed payments went up by 22%, and the leadership team wants a dashboard to actually understand what's going on — which payment methods are used most, when do failures happen, which customers bring in the most money, etc.

So my job was not just to "make some charts" — it was to actually dig into the transaction data and come up with real answers that a business team could use to take action.

## 2. About the dataset

I picked the UPI transactions dataset (`upi_transactions_2024.csv`). It has 5,020 transaction records with these details:

- transaction_id, user_id, timestamp
- payment_method (UPI, Card, Wallet, NetBanking, QR)
- amount, status (Success/Failed/Pending)
- city, merchant_category, device_type

500 unique users are in the dataset, so this is basically one full year of activity for 500 customers.

## 3. My approach — how I actually worked through this

I split my work into the 5 phases the task asked for. Here's what I genuinely did in each phase, and what I was thinking while doing it.

### Phase 1: Cleaning the data

First thing I did was just check `df.info()` and `df.isnull().sum()` to see how messy the data actually is. Found:
- 50 missing values in `amount`
- 30 missing values in `device_type`
- 20 duplicate `transaction_id` rows

For the missing amounts, I used **median** instead of mean. My reasoning: if I use mean, a few very big transactions (like someone paying ₹9,000 for something) would pull the "average" fill value up, and that wouldn't represent a typical transaction properly. Median is more realistic here.

For missing device_type, I just filled with the most common value (mode), since device type doesn't really have an obvious pattern to predict from other columns — Android was already the most used device by far, so filling with mode made sense.

Then I dropped the 20 duplicate transaction IDs. After all this cleaning, I was left with a clean dataset of **5,000 transactions**.

I also converted the `timestamp` column into an actual datetime (it was just text before), and pulled out `hour`, `day_name`, and `month_name` from it — this was important because a lot of my later analysis (like "when do failures spike") depends on having the hour available.

**Derived columns I created (as asked in the task):**
- `peak_hour_flag` — I decided peak hours are 10 AM–2 PM (lunch) and 6 PM–9 PM (evening), based on when people usually make purchases.
- `high_value_flag` — any transaction in the top 10% by amount (that came out to ≥ ₹2,990).
- `user_failure_rate` — for each user, what % of their transactions failed. I added this myself because I felt just knowing "when" failures happen isn't enough — knowing "who" has a high personal failure rate is also useful for the risk team (could indicate a bad card, bad network area, etc.)

### Phase 2: Designing the KPIs

The task gave me a list of mandatory KPIs to calculate: Total Transactions, Total Volume, Success Rate, Failure Rate, Avg Transaction Value, Peak Hour, Active Users, Revenue by Payment Method.

But I didn't want to just calculate them and dump numbers — I thought about **who actually uses each KPI**. So I grouped them into two buckets:

- **Payment Health KPIs** (Success Rate, Failure Rate, Avg Transaction Value) → these matter to the Operations/Risk team, since they tell you if the system is actually working properly.
- **Business KPIs** (Total Volume, Revenue by Method, Active Users) → these matter to the Product/Business team, since they tell you where money and growth are coming from.

This grouping is basically my answer to the task's question "which KPIs define payment health vs which help operations teams."

**Numbers I got (full detail in KPI_Report.md):**
- Total Transactions: 5,000
- Total Volume: ₹1.07 Crore approx
- Success Rate: 87.94%
- Failure Rate: 8.74%
- Avg Transaction Value: ₹2,139.89
- Peak Hour: 12 PM (noon)
- Active Users: 500 (all of them transacted at least once)

Honestly the 8.74% failure rate surprised me a bit — that means roughly 1 out of every 11-12 transactions is failing, which is a pretty big number for a payments company to ignore.

### Phase 3: Digging into patterns

This was the most interesting part for me. I looked at:

- **Payment method vs volume** → UPI dominates with 45.85% of total volume and 2,410 transactions, way ahead of Card (22.59%) and Wallet (16.88%). Makes sense given how popular UPI is in India right now.
- But then I checked **success rate by method** and found something I didn't expect — UPI's success rate (87.18%) is actually slightly *below* average, while NetBanking, which is barely used, has the *best* success rate (90.72%). So the most popular method isn't the most reliable one. I think this is genuinely one of my best findings in the whole project.
- **Time vs failure rate** → I grouped failure % by hour and found failures spike at 9 AM, 1 PM, and also late night between 12 AM–4 AM. My guess is these are transition points where system load changes suddenly (start of business hours, post-lunch rush, or maybe backend maintenance jobs running at night).
- **Device type vs usage** → Android leads by a big margin (3,307 transactions) over iOS (1,225) and Web (468), which is expected in the Indian market.
- **Merchant category vs revenue** → Fairly even split across Fuel, Healthcare, Shopping, Transport, etc. No single category is dominating, which tells me this dataset behaves like everyday household spending rather than one specific use-case (like just e-commerce).

I also made a simple funnel chart for Success → Pending → Failed to visually show how transactions "leak" at each stage.

### Phase 4: Grouping customers into segments

The task gave me example segment names (Heavy users, High-value users, etc.) but didn't tell me exactly how to define them, so I had to design my own logic. I used each user's:
- total number of transactions
- total amount spent

and split them using the 75th and 25th percentile as cutoffs. That gave me 5 segments:

1. Heavy + High-Value User
2. Heavy User
3. High-Value User
4. Regular/Frequent Transactor
5. Low-Engagement User

**What I found was honestly the biggest insight of the whole project:** the top 25% of users (Heavy + High-Value combined) bring in almost **half of all the revenue (~49.5%)**. Meanwhile, Low-Engagement users are the *biggest group by headcount* (30.4% of all users) but contribute the *least* revenue. This is a classic 80/20-type pattern and it's exactly the kind of thing a Product team would want to know — who to retain carefully vs who to try and re-engage.

### Phase 5: Building the dashboard

I built the interactive dashboard in Power BI with 4 sections, matching what the task asked for:
- **Executive Overview** — total volume, users, revenue, success rate at a glance
- **Transaction Dashboard** — payment trends over time
- **Customer Analytics Dashboard** — user segments and spending patterns
- **Risk Dashboard** — failed transactions, failure spikes, high-value transaction monitoring

I also exported the individual charts from Python (bar charts, funnel, pie chart for segments) into the `/images` folder so they can be viewed without opening Power BI.

## 4. Business questions — my answers

1. **Which payment methods perform best?** Depends on what "best" means — UPI wins on scale (most used, most volume), but NetBanking wins on reliability (highest success rate).
2. **Which time periods see the most failures?** Early morning (12 AM–4 AM) and two daily spikes around 9 AM and 1 PM.
3. **Which users contribute the most revenue?** Top 25% of users (Heavy + High-Value segments) generate about half of all revenue.
4. **Which regions need attention?** No city stood out as a huge problem area — Delhi and Mumbai had slightly lower transaction counts, might be worth a closer look, but nothing dramatic.
5. **What should product/ops teams actually do?** I've listed concrete recommendations below.

## 5. My recommendations

- Watch system load specifically around 9 AM, 1 PM, and 12–4 AM since that's when failures repeatedly spike.
- Look into why UPI and Wallet (the two most-used methods) have slightly lower success rates than NetBanking — even a small fix here would help a lot of transactions since they're used so much.
- Build a retention plan for the Heavy + High-Value users — losing even a handful of them would hurt revenue a lot since they're such a small group contributing so much.
- Run some kind of re-engagement campaign for Low-Engagement users, since they're the largest group and there's a lot of room to grow their activity.
- Start collecting a "reason for failure" field in future data, because right now I can tell *when* failures happen but not *why* — that limits how far the risk analysis can go.

## 6. Challenges I faced while doing this

- The task mentioned "complaints increased by 22%," but my dataset is only for one time period (2024), so I have no earlier data to actually compare against and verify that number. I decided to be upfront about this instead of making up a comparison.
- There's no fraud-related column in the dataset, so I could only analyze failed/pending transactions as a proxy for "risk," not actual fraud detection.
- Deciding where to draw the line for user segments (75th/25th percentile) was a judgment call on my part — a real business team might define "high value" differently, so I've mentioned this as a limitation rather than presenting it as the one correct answer.

## 7. What I learned from this project

- How to actually structure an analysis around *business questions* instead of just producing random charts.
- The difference between "popular" and "reliable" — UPI being the most used method doesn't mean it's the safest, which was a good reminder to not assume popularity = quality.
- How to design my own segmentation logic when the task doesn't give exact rules, and how to justify the choices I made.
- How important it is to be honest about the limitations of a dataset instead of overselling the conclusions — like being upfront that I couldn't verify the 22% complaint increase.

## 8. Conclusion

Overall, this project gave me a proper, realistic view of what a Data Analyst does at a digital payments company — cleaning messy data, designing KPIs that different teams actually care about, finding patterns in failures and revenue, and turning all of that into clear business recommendations. The biggest takeaways for me were the reliability gap in UPI/Wallet versus NetBanking, and the revenue concentration in the top user segment — both are the kind of insights a real business would act on.

---
*This report is written in my own words based on my analysis of the dataset — full KPI numbers are documented separately in KPI_Report.md, and the technical steps are in the Jupyter notebook.*
