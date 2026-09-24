# TELECOM CUSTOMER CHURN ANALYSIS USING POWER BI
## A Power BI project Telecom Customer churn, service performance & Degradation Tracker Analysis.

---

## Table of contents
1 Project Overview
2 Dataset Overview
3 Database Structure & Relationship
4 Data Quality Assessment
5 Data Cleaning & Error Correction
6 Feature Engineering
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
In large-scale telecommunications environments, operational metrics frequently fragment across billing ledgers, network telemetry routers, and customer service ticket databases. This fragmentation makes it incredibly difficult to see where technical service level agreement (SLA) defaults are actively hurting corporate finances and driving customer cancellations.

The objective of this business intelligence project was to engineer an end-to-end, multi-page Power BI dashboard application that unifies these disconnected transaction logs into a cohesive executive diagnostic platform. By connecting customer financial performance directly to technical network data, this project provides senior leadership with a self-service tool to minimize revenue losses and proactively target high-risk accounts before they cancel their service.

---

## 2 Dataset Overview
The relational warehouse structure utilized for this analysis mimics an enterprise telecom operations platform, consisting of 4 distinct tables:
*   Customer_Directory: Master lookup database capturing customer profiles, unique identifiers, signup dates, contract classifications, and churn indicators.
*   Billing_Revenue: Financial ledger containing transactional line entries tracking monthly billing targets, payment dates, and active collection statuses.
*   Service_Performance: Network engineering telemetry tracking data usage metrics, dropped call incidents, and operational network latency times.
*   Support_Tickets: Customer operations incident logs tracking customer complaints, resolution durations, and ticket classification topics.

---

## 3 Database Structure & Relationship
To ensure fast reporting performance and prevent common cross-table calculation errors, I designed a centralized Star Schema data model inside the Power BI engine. A strict 1-to-Many plumbing layout was established, drawing connection lines from the central master parent dimension lookup table down to all child transactional fact tables:

*   Customer_Directory[CustomerID] filtering Billing_Revenue[CustomerID]
*   Customer_Directory[CustomerID] filtering Service_Performance[CustomerID]
*   Customer_Directory[CustomerID] filtering Support_Tickets[CustomerID]

---

## 4 Data Quality Assessment
Before building the analytical charts, a thorough data quality audit exposed critical tracking vulnerabilities that threatened to distort executive reporting and break database calculations:
*   Severe Date Contamination: The SignupDate column arrived deeply corrupted, featuring an unpredictable mix of regional text structures (some records reading YYYY/MM/DD, others reading DD/MM/YYYY, and others reading MM/DD/YYYY) that completely broke standard calendar lookups.
*   Fragmented Data Categories: Manual data entry typos in the network type column artificially split identical data rows into separate categories like 4G and 4G LTE.
*   Visual Format Glitches: Trailing decimal errors and blank fields cut off system labels, crowding out visual padding across dashboard grids.
*   Axis Range Gaps: Incomplete data keys generated broken segments on line graphs, cutting off the continuous timelines required for executive trend tracking.

---

## 5 Data Cleaning & Error Correction
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

## 6 Feature Engineering
Rather than using system-slowing grid lookups, the cleaned tables were integrated into a high-performance Star Schema Relational Model inside Power BI. To compute precise financial and operational metrics, I coded a series of custom Data Analysis Expressions (DAX) measures as pure, plain text:

Total Revenue = SUM(Billing_Revenue[MonthlyBill_USD])

Unpaid Revenue = CALCULATE([Total Revenue], Billing_Revenue[PaymentStatus] = "Unpaid")

Overdue Revenue = CALCULATE([Total Revenue], Billing_Revenue[PaymentStatus] = "Overdue")

Total Tickets = COUNT(Support_Tickets[TicketID])

Churn Rate % = DIVIDE(CALCULATE(COUNT(Customer_Directory[CustomerID]), Customer_Directory[Churned] = "Yes"), COUNT(Customer_Directory[CustomerID]), 0) + 0

By engineering the + 0 fallback addition right at the end of the Churn Rate calculation, I forced the model to explicitly display 0.00% instead of empty blanks, restoring complete visual balance to our corporate matrix grids.

---

## 7 Total Revenue, Dataused_GB by Network type
By mapping our custom Total Revenue measure alongside the raw DataUsed_GB column across network generations, the system generated a side-by-side clustered dual-column chart. This analysis explicitly exposes where user network traffic demands are heaviest compared to financial returns, helping infrastructure teams allocate technical capacity accurately.

---

## 8 Overdue Revenue by complaint category
To isolate the exact financial impact of customer service drops, the Overdue Revenue DAX measure was plotted against text complaint categories inside a customized donut visual. This maps our outstanding capital risk directly to customer friction points, proving that technical infrastructure faults cause direct payment blockages on the ledger.

---

## 9 Contract type, Churn rate% Total Revenue
To evaluate customer retention risks, a deep-dive corporate matrix grid was deployed to group contract structures directly against churn metrics. This visual isolates high-risk accounts instantly while fixing trailing decimal errors to present a clean, boardroom-ready layout.

---

## 10 Total tickets count
To maintain a continuous high-level health check on operational performance, a dedicated executive KPI scorecard block was engineered using the Total Tickets DAX measure. The category labels were unchecked and replaced with custom bold formatting to display the total customer incident volume cleanly.

---

## 11 Sign up date, sum of dropped call count and sum of Avg Latency_ms
To audit technical infrastructure health chronologically, dropped call frequencies and average latency speeds were plotted together across a continuous chronological axis. By unfreezing series locks inside the lines properties menu and formatting the paths to bold high-contrast white and yellow, this dual-line graph shows executives exactly when customer service quality degraded over time.

---

## 12 Key Findings & Insights
*   The Retention Core: A high 66.00% Churn Rate was isolated exclusively within Month-to-Month contracts. Annual agreements maintained a perfect 0.00% customer drop rate, showing that flexible plans present the highest risk to business stability.
*   The Financial Leak Link: Customer support tickets are directly tied to cash flow leaks. Billing Issues and Network Outages generate a massive 80% of all customer complaints, directly causing a $7.98K bottleneck in Overdue Revenue and locking up $9.32K in Unpaid Bills.
*   The Technical SLA Bottleneck: Legacy 3G network assets are heavily degrading, showing severe latency spikes and high dropped call frequencies compared to optimized 4G and 5G connections.

---

## 13 Recommendations
1.  Deploy Targeted Retention Programs: Shift high-risk Month-to-Month accounts onto stable annual agreements by introducing proactive pricing incentives before their next billing loop.
2.  Decommission Legacy Infrastructure: Fast-track the migration of remaining 3G infrastructure users onto active 4G and 5G platforms to resolve the root cause of outages and dropped calls.
3.  Prioritize Revenue Recovery Outreach: Direct customer collections tracking explicitly toward accounts flagged under Billing Issues to clear the $7.98K overdue bottleneck.

---

## 14 Conclusion
