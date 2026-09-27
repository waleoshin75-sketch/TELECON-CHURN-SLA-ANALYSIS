# Telecom Customer Churn Analysis Using Power BI

## A Power BI Project: Telecom Customer Churn, Service Performance and Degradation Tracker Analysis

---

## Table of Contents

- [1 Project Overview](#1-project-overview)
- [2 Dataset Overview](#2-dataset-overview)
- [3 Data Quality Assessment](#3-data-quality-assessment)
- [4 Data Cleaning & Error Correction](#4-data-cleaning--error-correction)
- [5 Feature Engineering](#5-feature-engineering)
- [6 Data Modeling & Relational Schema](#6-data-modeling--relational-schema)
- [7 Total Revenue, DataUsed_GB by Network Type](#7-total-revenue-dataused_gb-by-network-type)
- [8 Overdue Revenue by Complaint Category](#8-overdue-revenue-by-complaint-category)
- [9 Contract Type, Churn Rate % and Total Revenue](#9-contract-type-churn-rate-total-revenue)
- [10 Total Tickets Count](#10-total-tickets-count)
- [11 Signup Date, Dropped Call Count and Average Latency](#11-sign-up-date-sum-of-dropped-call-count-and-sum-of-avg-latency_ms)
- [12 Key Findings & Insights](#12-key-findings--insights)
- [13 Recommendations](#13-recommendations)
- [14 Conclusion](#14-conclusion)

---

## 1 Project Overview

In large telecom operations, billing systems, network telemetry, and customer service logs tend to live in separate silos. That separation makes it hard to see, in one place, where technical service failures are quietly draining revenue and pushing customers toward the door.

This project builds a single Power BI dashboard that pulls those three worlds together. Billing, network performance, and support tickets are linked directly to the same customer, so leadership can trace a dropped call or a network outage all the way through to unpaid revenue and churn risk, rather than reading three disconnected reports and guessing at the connection.

---

## 2 Dataset Overview

The model is built on four related tables, mirroring how a real telecom operator's data warehouse is typically structured.

| Table | What It Holds |
| --- | --- |
| Customer_Directory | Master customer profiles, signup dates, contract type, and churn status |
| Billing_Revenue | Monthly billing amounts, payment dates, and payment status |
| Service_Performance | Data usage, dropped call counts, and network latency |
| Support_Tickets | Customer complaints, resolution time, and ticket category |

Customer_Directory sits at the center as the single source of truth for who each customer is; the other three tables each describe a different slice of that customer's experience.

---

## 3 Data Quality Assessment

Before any chart could be trusted, the raw data needed a proper audit. A few issues stood out immediately:

- **Mixed date formats:** SignupDate mixed three different regional layouts in the same column, YYYY/MM/DD, DD/MM/YYYY, and MM/DD/YYYY, which broke any standard date conversion.
- **Duplicate categories from typos:** Manual entry errors split what should have been one network type into two, 4G and 4G LTE, artificially inflating the category count.
- **Formatting noise:** Trailing decimals and blank labels were crowding out visual space across the dashboard grids.
- **Broken trend lines:** Gaps in the underlying data keys were cutting continuous timelines into disconnected segments on the line charts.

---

## 4 Data Cleaning & Error Correction

Two fixes did most of the heavy lifting here, both handled inside Power Query before the model was ever loaded.

### A. Unifying Mixed Date Formats

Since the three date formats in SignupDate couldn't be resolved by a single built-in converter, a custom M function parses each value by structure. If the first segment is four digits long, it's read as year first. Otherwise, if the first number is greater than twelve, it can't be a month, so it's read as day first. Everything else falls back to month first, the most common default.

```m
try
    let
        Parts = Text.Split([SignupDate], "/"),
        P1 = Number.FromText(Parts{0}),
        P2 = Number.FromText(Parts{1}),
        P3 = Number.FromText(Parts{2})
    in
        if Text.Length(Parts{0}) = 4 then
            #date(P1, P2, P3)
        else if P1 > 12 then
            #date(P3, P2, P1)
        else
            #date(P3, P1, P2)
otherwise
    null
```

### B. Merging Duplicate Categories

The stray "4G LTE" label was collapsed into the standard "4G" value using a simple find-and-replace transformation in Power Query. That one fix took the network type chart from four confusing bars down to the three clean generations it should have shown from the start.

---

## 5 Feature Engineering

Rather than relying on slow implicit aggregations, five DAX measures were written explicitly to drive every visual in the report.

```dax
Total Revenue = SUM(Billing_Revenue[MonthlyBill_USD])

Unpaid Revenue = CALCULATE([Total Revenue], Billing_Revenue[PaymentStatus] = "Unpaid")

Overdue Revenue = CALCULATE([Total Revenue], Billing_Revenue[PaymentStatus] = "Overdue")

Total Tickets = COUNT(Support_Tickets[TicketID])

Churn Rate % = 
DIVIDE(
    CALCULATE(COUNT(Customer_Directory[CustomerID]), Customer_Directory[Churned] = "Yes"),
    COUNT(Customer_Directory[CustomerID]),
    0
) + 0
```

The trailing "+ 0" on Churn Rate % is a small but deliberate fix: it forces DAX to display a genuine 0.00 percent instead of a blank cell whenever a segment has no churned customers, keeping every card and matrix visually consistent.

---

## 6 Data Modeling & Relational Schema

The four tables were connected in a star schema, with Customer_Directory as the single dimension table and the other three as fact tables joined to it one-to-many on CustomerID.

- Customer_Directory\[CustomerID\] to Billing_Revenue\[CustomerID\]
- Customer_Directory\[CustomerID\] to Service_Performance\[CustomerID\]
- Customer_Directory\[CustomerID\] to Support_Tickets\[CustomerID\]

This keeps every fact table filtering independently off the same customer record, so a slicer on contract type or churn status correctly filters billing, network, and support data all at once.

![Data model diagram showing Customer_Directory connected to Billing_Revenue, Service_Performance, and Support_Tickets](https://github.com/user-attachments/assets/d58ff5b5-5820-45dd-b9fb-5b6942493520)

---

## 7 Total Revenue, DataUsed_GB by Network Type

A clustered column chart plots Total Revenue alongside raw data usage for each network generation. Placing both metrics side by side shows immediately where network demand and financial return are out of step, which is exactly the kind of signal infrastructure teams need when deciding where to invest capacity.

---

## 8 Overdue Revenue by Complaint Category

A donut chart breaks down Overdue Revenue by complaint category, connecting unresolved technical issues directly to unpaid balances. It makes a case that's easy to miss in isolated reports: service friction isn't just a support problem, it's actively blocking cash collection.

---

## 9 Contract Type, Churn Rate % and Total Revenue

A matrix visual cross-tabulates contract type against Churn Rate % and Total Revenue, surfacing which contract structures carry the most retention risk at a glance. Trailing decimal formatting was cleaned up here specifically, so the table reads cleanly in a boardroom setting rather than looking like a raw data export.

---

## 10 Total Tickets Count

A single KPI card displays the Total Tickets measure as a running health check on support volume. Category labels were switched off in favor of bold custom formatting, keeping the card focused on the one number that matters at a glance.

---

## 11 Signup Date, Dropped Call Count and Average Latency

Dropped call counts and average latency are plotted together on a shared timeline, unlocked from their default axis pairing so both series render clearly in bold, high contrast colors. Seeing both metrics move together over time makes it possible to pinpoint exactly when network quality started to slip, rather than noticing the decline only after complaints pile up.

---

## 12 Key Findings & Insights

- **Contract type is the clearest churn signal in the data.** Month-to-month customers churned at 66.00 percent, while customers on annual contracts churned at 0.00 percent. Contract length alone separates the highest-risk customers from the safest ones.
- **Support tickets and revenue leakage are directly linked.** Billing issues and network outages together account for 80 percent of all complaints, and trace directly to 7.98 thousand dollars in overdue revenue and 9.32 thousand dollars in unpaid bills.
- **Legacy network infrastructure is an operational bottleneck.** Remaining 3G connections show noticeably higher latency and more dropped calls than 4G and 5G, tying older infrastructure directly to the service failures driving those complaints.

---

## 13 Recommendations

1. **Migrate high-risk accounts off month-to-month plans.** Offer proactive pricing incentives to move month-to-month customers onto annual contracts ahead of their next billing cycle.
2. **Retire remaining 3G infrastructure.** Prioritize migrating the last 3G users onto 4G or 5G to address dropped calls and latency at the source, rather than continuing to handle the complaints they generate.
3. **Target collections at billing-related complaints first.** Focus recovery outreach on accounts flagged under billing issues, where the 7.98 thousand dollar overdue balance is concentrated.

---

## 14 Conclusion

This project turned three disconnected telecom data sources into one coherent diagnostic tool. Fixing the underlying date and category errors in Power Query, building a proper star schema, and writing explicit DAX measures made it possible to trace a technical failure all the way through to its financial impact, giving leadership a clear, evidence backed starting point for reducing churn and recovering revenue.
