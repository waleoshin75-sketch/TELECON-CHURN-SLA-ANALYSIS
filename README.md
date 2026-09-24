# TELECOM CUSTOMER CHURN ANALYSIS USING POWER BI
## A Power BI project Telecom Customer churn, service performance & Degradation Tracker Analysis.

---

## Table of contents
1 Project Overview
2 Dataset Overview
3 Data Quality Assessment
4 Data Cleaning & Error Correction
5 Feature Engineering
6 Data Modeling & Relational Schema
7 Total Revenue, Dataused_GB by Network type
8 Overdue Revenue by complaint category
9 Contract type, Churn rate% Total Revenue
10 Total tickets count
11 Sign up date, sum of dropped call count and sum of Avg Latency_ms
12 Key Findings & Insights
13 Recommendations
14 Conclusion

---

## 1 Project Overview
In modern, large-scale telecommunications environments, operational data frequently fragments across billing registries, network telemetry routers, and customer service incident databases. This fragmentation makes it difficult to see where technical service level agreement (SLA) defaults are actively hurting corporate cash flow and driving customer cancellations.

The objective of this business intelligence project was to engineer an end-to-end, multi-page Power BI dashboard application that unifies these disconnected logs into a cohesive executive diagnostic platform. By connecting customer financial performance directly to technical network data, this project provides senior leadership with a self-service tool to minimize revenue losses and proactively target high-risk accounts before they cancel their service.

---

## 2 Dataset Overview
The relational warehouse structure utilized for this analysis mimics an enterprise telecom operations platform, consisting of 4 distinct tables:
*   Customer_Directory: Master lookup database capturing customer profiles, unique identifiers, signup dates, contract classifications, and churn indicators.
*   Billing_Revenue: Financial ledger containing transactional line entries tracking monthly billing targets, payment dates, and active collection statuses.
*   Service_Performance: Network engineering telemetry tracking data usage metrics, dropped call incidents, and operational network latency times.
*   Support_Tickets: Customer operations incident logs tracking customer complaints, resolution durations, and ticket classification topics.

---

## 3 Data Quality Assessment
Before building the analytical charts, a thorough data quality audit exposed critical tracking vulnerabilities that threatened to distort executive reporting and break database calculations:
*   Severe Date Contamination: The SignupDate column arrived deeply corrupted, featuring an unpredictable mix of regional text structures (some records reading YYYY/MM/DD, others reading DD/MM/YYYY, and others reading MM/DD/YYYY) that completely broke standard calendar lookups.
*   Fragmented Data Categories: Manual data entry typos in the network type column artificially split identical data rows into separate categories like 4G and 4G LTE.
*   Visual Format Glitches: Trailing decimal errors and blank fields cut off system labels, crowding out visual padding across dashboard grids.
*   Axis Range Gaps: Incomplete data keys generated broken segments on line graphs, cutting off the continuous timelines required for executive trend tracking.

---

## 4 Data Cleaning & Error Correction
To restore complete data integrity before connecting the database model, the following advanced programmatic overrides were deployed inside the Power Query ETL engine:

### A. Advanced Date Unification via Custom M-Code
*   Problem: Mixed regional date layouts caused standard text-to-date converters to drop records or misread days as months.
*   Solution: I wrote a custom conditional M-code formula. By implementing list point indicators, the programming steps are forced onto clean, distinct lines as pure text:

try let Parts = Text.Split([SignupDate], "/"), P1 = Number.FromText(Parts{0}), P2 = Number.FromText(Parts{1}), P3 = Number.FromText(Parts{2}) in if Text.Length(Parts{0}) = 4 then #date(P1, P2, P3) else if P1 > 12 then #date(P3, P2, P1) else #date(P3, P1, P2) otherwise null


### B. Categorical Clean-Up and Merge
*   Problem: The duplicate typo 4G LTE forced our revenue charts to show four vertical bars instead of three clean generations.
*   Solution: Deployed an ETL text replacement routine inside Power Query, targeting the exact string values to merge all 4G LTE rows into the standard 4G text value, which restored immediate data consistency.

---

## 5 Feature Engineering
Rather than using system-slowing lookup configurations, I coded custom Data Analysis Expressions (DAX) measures as pure, plain text to build calculated metrics across the warehouse:

Total Revenue = SUM(Billing_Revenue[MonthlyBill_USD])

Unpaid Revenue = CALCULATE([Total Revenue], Billing_Revenue[PaymentStatus] = "Unpaid")

Overdue Revenue = CALCULATE([Total Revenue], Billing_Revenue[PaymentStatus] = "Overdue")

Total Tickets = COUNT(Support_Tickets[TicketID])

Churn Rate % = DIVIDE(CALCULATE(COUNT(Customer_Directory[CustomerID]), Customer_Directory[Churned] = "Yes"), COUNT(Customer_Directory[CustomerID]), 0) + 0

By engineering the + 0 fallback addition right at the end of the Churn Rate calculation, I forced the model to explicitly display 0.00% instead of empty blanks, restoring complete visual balance to our corporate matrix grids.

---

## 6 Data Modeling & Relational Schema
To establish an enterprise data warehouse structure, the cleaned tables were integrated into a centralized Star Schema data model inside Power BI's database diagram view. A strict 1-to-Many relational plumbing layout was established, drawing connection lines from the central master parent dimension lookup table down to all child transactional fact tables:

*   Customer_Directory[CustomerID] Connected to Billing_Revenue[CustomerID]
*   Customer_Directory[CustomerID] Connected to Service_Performance[CustomerID]
*   Customer_Directory[CustomerID] Connected to Support_Tickets[CustomerID]
<img width="952" height="494" alt="image" src="https://github.com/user-attachments/assets/1b1297ae-e249-4800-a1ff-721bf6ddcca1" />
