# Telecom Customer Churn Analytics Using Power BI

**Power Query M | DAX | Star schema | Churn, revenue leakage and service degradation tracking**

Unified billing, network and support data into one Power BI model that links service failures to unpaid revenue and customer churn.

| | |
|---|---|
| **Tools** | Power BI Desktop, Power Query (M), DAX |
| **Data** | 4 related tables (1 dimension, 3 fact tables) |
| **Deliverable** | Multi-page dashboard with five reusable DAX measures |

---

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [Dataset Overview](#2-dataset-overview)
3. [Data Quality Assessment](#3-data-quality-assessment)
4. [Data Cleaning and Error Correction](#4-data-cleaning-and-error-correction)
5. [Feature Engineering and DAX](#5-feature-engineering-and-dax)
6. [Data Modeling](#6-data-modeling)
7. [Dashboard Analysis](#7-dashboard-analysis)
8. [Key Findings and Insights](#8-key-findings-and-isnights)
9. [Recommendations](#9-recommendations)
10. [Conclusion](#10-conclusion)

---

## 1. Project Overview
Telecom data typically sits in separate billing, network and support systems, which hides how service failures hurt revenue and retention. This project connects the three to a single customer record so leadership can trace a technical fault through to unpaid bills and churn risk.

## 2. Dataset Overview

| Table | Type | Contents |
|---|---|---|
| `Customer_Directory` | Dimension | Customer profile, signup date, contract type, churn status |
| `Billing_Revenue` | Fact | Monthly bill, invoice, payment status |
| `Service_Performance` | Fact | Data usage, dropped calls, latency, network type |
| `Support_Tickets` | Fact | Complaint category, resolution time, ticket ID |

## 3. Data Quality Assessment

| Issue | Impact |
|---|---|
| `SignupDate` mixed three formats (YYYY/MM/DD, DD/MM/YYYY, MM/DD/YYYY) | Standard date conversion failed or misread days as months |
| Typo split one network type into "4G" and "4G LTE" | Revenue chart showed four bars instead of three generations |
| Trailing decimals and blank labels | Cluttered matrix and card visuals |
| Gaps in date keys | Broken segments on trend lines |

## 4. Data Cleaning and Error Correction

**Mixed date formats:** a custom Power Query column reads each value by structure. A four-digit first segment means year first. A first number above 12 cannot be a month, so it is day first. Anything else falls back to month first. Unreadable values return null instead of breaking the query.

```powerquery
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

**Duplicate category:** a Replace Values step merged "4G LTE" into "4G", restoring three network generations.

## 5. Feature Engineering and DAX

```powerquery
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

The trailing `+ 0` in `Churn Rate %` forces a displayed 0.00 percent instead of a blank cell for segments with no churn, keeping matrix visuals consistent.

## 6. Data Modeling

Star schema with `Customer_Directory` as the single dimension and three one-to-many relationships on `CustomerID`. All three relationships are active.

| From (many) | To (one) |
|---|---|
| `Billing_Revenue[CustomerID]` | `Customer_Directory[CustomerID]` |
| `Service_Performance[CustomerID]` | `Customer_Directory[CustomerID]` |
| `Support_Tickets[CustomerID]` | `Customer_Directory[CustomerID]` |

<img width="959" height="503" alt="Power BI model view showing Customer_Directory connected to Billing_Revenue, Service_Performance and Support_Tickets" src="https://github.com/user-attachments/assets/d58ff5b5-5820-45dd-b9fb-5b6942493520" />

## 7. Dashboard Analysis

| Visual | Measures | Purpose |
|---|---|---|
| Clustered column | Total Revenue, DataUsed_GB by network type | Compare traffic demand against revenue |
| Donut | Overdue Revenue by complaint category | Link service friction to unpaid balances |
| Matrix | Contract type, Churn Rate %, Total Revenue | Identify high-risk contract segments |
| KPI card | Total Tickets | Support volume at a glance |
| Dual line | Dropped calls and average latency by signup date | Show when service quality degraded |

### Dashboard Preview

**Executive KPIs**

<!-- Drag telecom-executive-kpis.png here while editing this file on GitHub; GitHub will insert the image link. Alt text: Telecom Customer Churn Executive KPIs dashboard -->

Total revenue is $44,229 with a 33.17% churn rate. Month-to-month contracts churn at 66.00%.

**Service Performance and SLA Degradation Tracker**

<!-- Drag telecom-business-performance.png here while editing this file on GitHub; GitHub will insert the image link. Alt text: Telecom Business Performance Analysis dashboard with revenue by network type, overdue revenue by complaint category and contract churn -->


## 8. Key Findings

| Finding | Detail |
|---|---|
| Contract type drives churn | Month-to-Month churn was 66.00 percent, while annual contracts churned at 0.00 percent |
| Support issues link to cash flow | Billing Issues and Network Outages made up 80 percent of complaints, tied to 7.98 thousand dollars overdue and 9.32 thousand dollars unpaid |
| Legacy network is the bottleneck | 3G connections showed higher latency and more dropped calls than 4G and 5G |

## 9. Recommendations
1. **Move Month-to-Month customers to annual plans:** offer pricing incentives ahead of the next billing cycle.
2. **Retire remaining 3G infrastructure:** migrate users to 4G or 5G to remove the source of outages and dropped calls.
3. **Target collections at billing complaints:** focus recovery on the accounts behind the 7.98 thousand dollar overdue balance.

## 10. Conclusion
Fixing the date and category errors in Power Query, modeling the tables as a star schema and writing explicit DAX measures made it possible to trace service failures to their financial impact.

