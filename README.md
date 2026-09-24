# TELECOM CUSTOMER CHURN ANALYSIS USING POWER BI
## A Power BI project Telecom Customer churn, service performance & Degradation Tracker Analysis.
### Consumer Electronics Retail – Synthetic Electronics Dataset (2026)

---

## Table of contents
1 Project Overview
2 Dataset Overview
3 Data Quality Assessment
4 Data Cleaning & Error Correction
5 Feature Engineering
6 Revenue By Customer Analysis
7 Revenue By Category Analysis
8 Revenue By Region Analysis
9 Key Findings & Insights
10 Recommendations
11 Conclusion

---

## 1 Project Overview
In modern, large-scale telecommunications environments, operational data frequently fragments across billing ledgers, network telemetry routers, and customer service ticket databases. This fragmentation makes it difficult to see where technical service degradation is directly hurting company finances and driving customer cancellations.

The objective of this corporate business intelligence project was to engineer an end-to-end, multi-page Power BI application that transforms disconnected transaction logs into a cohesive executive diagnostic platform. By connecting customer financial data directly to technical service level agreements (SLAs), this project provides senior leadership with a self-service tool to minimize financial losses and proactively target high-risk accounts before they cancel their service.

---

## 2 Dataset Overview
The relational database structure utilized for this analysis mimics an enterprise telecom operations environment, consisting of 4 distinct tables:
*   Customer_Directory: Master lookup database capturing customer profiles, unique identifiers, signup dates, contract classifications, and churn indicators.
*   Billing_Revenue: Financial ledger containing transactional line entries tracking monthly billing targets, payment dates, and active collection statuses.
*   Service_Performance: Network engineering telemetry tracking data usage metrics, dropped call incidents, and operational network latency times.
*   Support_Tickets: Customer operations incident logs tracking customer complaints, resolution durations, and ticket classification topics.

---

## 3 Data Quality Assessment
Before building the analytical models, a thorough data quality audit exposed critical tracking vulnerabilities that threatened to distort executive reporting and break database calculations:
*   Severe Date Contamination: The SignupDate column arrived deeply corrupted, featuring an unpredictable mix of regional text structures (some records reading YYYY/MM/DD, others reading DD/MM/YYYY, and others reading MM/DD/YYYY) that completely broke standard calendar lookups.
*   Fragmented Data Categories: Manual data entry typos in the network type column artificially split identical data rows into separate categories like 4G and 4G LTE.
*   Visual Format Glitches: Trailing decimal errors and blank fields cut off system labels, crowding out visual padding across dashboard grids.
*   Axis Range Gaps: Incomplete data keys generated broken segments on line graphs, cutting off the continuous timelines required for executive trend tracking.

---

## 4 Data Cleaning & Error Correction
To restore complete data integrity before connecting the database model, the following advanced programmatic overrides were deployed inside the Power Query ETL engine:

### A. Advanced Date Unification via Custom M-Code
*   Problem: Mixed regional date layouts caused standard text-to-date converters to drop records or misread days as months.
*   Solution: I wrote a custom conditional M-code formula to systematically split the string components, parse their textual length, and evaluate numerical boundaries to force unified calendar outputs as pure text without code container tags:

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

### B. Categorical Clean-Up and Merge
*   Problem: The duplicate typo 4G LTE forced our revenue charts to show four vertical bars instead of three clean generations.
*   Solution: Deployed an ETL text replacement routine inside Power Query, targeting the exact string values to merge all 4G LTE rows into the standard 4G text value, which restored immediate data consistency.

---

## 5 Feature Engineering & Relational Modeling
Rather than using system-slowing lookups, the cleaned tables were integrated into a high-performance Star Schema Relational Model inside Power BI using a 1-to-Many plumbing layout. To compute precise financial and operational metrics, I coded a series of custom Data Analysis Expressions (DAX) measures as pure, plain text:

Total Revenue = SUM(Billing_Revenue[MonthlyBill_USD])

Unpaid Revenue = CALCULATE([Total Revenue], Billing_Revenue[PaymentStatus] = "Unpaid")

Overdue Revenue = CALCULATE([Total Revenue], Billing_Revenue[PaymentStatus] = "Overdue")

Total Tickets = COUNT(Support_Tickets[TicketID])

Churn Rate % = DIVIDE(CALCULATE(COUNT(Customer_Directory[CustomerID]), Customer_Directory[Churned] = "Yes"), COUNT(Customer_Directory[CustomerID]), 0) + 0

By engineering the + 0 fallback addition right at the end of the Churn Rate calculation, I forced the model to explicitly display 0.00% instead of empty blanks, restoring complete visual balance to our corporate matrix grids.

---

## 6 Revenue By Customer Analysis
By mapping our custom financial measures against master buyer profiles, the system generated an active accounts tracker. This analysis isolates customer valuation concentrations, allowing executives to see exactly how much capital is locked up in active unpaid invoices versus overdue default risks.

---

## 7 Revenue By Category Analysis
To audit product line capacity, total billing values were contrasted side-by-side with data usage volumes across each network generation tier. This evaluation exposes where network demands are heaviest compared to financial returns.

---

## 8 Revenue By Region Analysis
Connecting customer geographic profiles directly to our operational metrics allows us to evaluate regional health. By using basic filtering overrides on our date parameters, I eliminated layout gaps, allowing the multi-axis trend lines to stretch across the page canvas to show precisely when network drops and latency spikes fluctuated over time.

---

## 9 Key Findings & Insights
*   The Retention Core: A high 66.00% Churn Rate was isolated exclusively within Month-to-Month contracts. Annual agreements maintained a perfect 0.00% customer drop rate, showing that flexible plans present the highest risk to business stability.
*   The Financial Leak Link: Customer support tickets are directly tied to cash flow leaks. Billing Issues and Network Outages generate a massive 80% of all customer complaints, directly causing a $7.98K bottleneck in Overdue Revenue and locking up $9.32K in Unpaid Bills.
*   The Technical SLA Bottleneck: Legacy 3G network assets are heavily degrading, showing severe latency spikes and high dropped call frequencies compared to optimized 4G and 5G connections.

---

## 10 Recommendations
1.  Deploy Targeted Retention Programs: Shift high-risk Month-to-Month accounts onto stable annual agreements by introducing proactive pricing incentives before their next billing loop.
2.  Decommission Legacy Infrastructure: Fast-track the migration of remaining 3G infrastructure users onto active 4G and 5G platforms to resolve the root cause of outages and dropped calls.
3.  Prioritize Revenue Recovery Outreach: Direct customer collections tracking explicitly toward accounts flagged under Billing Issues to clear the $7.98K overdue bottleneck.

---

## 11 Conclusion
This project shows how advanced business intelligence architecture can convert messy operational logs into a powerful tool for customer retention and revenue management. By implementing custom M-code date parsing, establishing an optimized Star Schema relational warehouse model, and building tailored DAX measures, this application provides telecom executives with the exact insights needed to optimize infrastructure investments, resolve customer pain points, and secure corporate cash flow.
