# Power BI Data Analysis Report & Documentation

## 1. Executive Summary

This repository contains the comprehensive analytical documentation and structural overview of the **Power BI Analytics Solution**. The project consolidates multi-source enterprise data to deliver actionable insights across sales performance, financial health, operational efficiency, and customer behavior. 

Key outcomes documented in this report include:
- **Data Model Architecture**: Relational star-schema design optimized for analytical DAX computations.
- **KPI Dashboards**: Dynamic visual report pages providing high-level executive summaries and drill-down operational metrics.
- **Data Pipeline & Governance**: Automated data refresh workflows, DAX measure definitions, and Row-Level Security (RLS) policies.

---

## 2. Power BI Architecture & Metadata Overview

The report infrastructure is built upon Power BI Desktop and Service framework components, utilizing custom themes, data model relationships, and standardized layouts.

| Component | Specification / Artifact | Description |
| :--- | :--- | :--- |
| **Data Model Engine** | VertiPaq In-Memory Engine | Columnar storage for maximum compression and fast query speed |
| **Theme & Styling** | `CY26SU07.json` / Custom Shared Resources | Standardized corporate branding palette with accessible typography |
| **Security Architecture** | Role-Based RLS (`SecurityBindings`) | Enforces data privacy across geographic regions and business units |
| **Report Layout** | Multi-Tab Interactive Visual Canvas | Optimized responsive layout for desktop, tablet, and mobile viewing |

---

## 3. Data Model & Schema Design

The underlying data model follows a robust **Star Schema** approach, separating quantitative transactional facts from descriptive dimension tables to ensure high query performance and scalable modeling.

```
                  +-----------------------+
                  |    Dim_Customer       |
                  +-----------------------+
                              |
                              | 1:N
                              v
+------------------+     +-----------------------+     +------------------+
|    Dim_Date      |---->|      Fact_Sales       |<----|   Dim_Product    |
+------------------+ 1:N +-----------------------+ 1:N +------------------+
                              ^
                              | 1:N
                  +-----------------------+
                  |    Dim_Geography      |
                  +-----------------------+
```

### Table Definitions & Key Attributes

1. **`Fact_Sales` (Fact Table)**
   - `SalesID` [PK]: Unique transaction identifier
   - `DateKey` [FK]: Joins to `Dim_Date`
   - `CustomerKey` [FK]: Joins to `Dim_Customer`
   - `ProductKey` [FK]: Joins to `Dim_Product`
   - `GeographyKey` [FK]: Joins to `Dim_Geography`
   - `SalesAmount`: Total gross revenue per line item
   - `OrderQuantity`: Units sold
   - `DiscountAmount`: Total promotional discount applied
   - `TotalCost`: Unit Cost * Order Quantity

2. **`Dim_Customer` (Dimension Table)**
   - `CustomerKey` [PK]: Unique customer ID
   - `CustomerName`: Full name of the account/individual
   - `CustomerSegment`: Enterprise, SMB, Consumer
   - `CustomerStatus`: Active, Churned, Lead

3. **`Dim_Product` (Dimension Table)**
   - `ProductKey` [PK]: Unique product SKU
   - `ProductName`: Descriptive item name
   - `Category`: High-level product group
   - `SubCategory`: Granular classification
   - `UnitPrice`: Base listing price
   - `UnitCost`: COGS per unit

4. **`Dim_Date` (Calendar Dimension)**
   - `DateKey` [PK]: Date formatted as YYYYMMDD
   - `Date`: Calendar date
   - `Year`, `Quarter`, `Month`, `MonthName`, `DayOfWeek`
   - `IsFiscalYear`, `FiscalQuarter`

---

## 4. Key Metrics & DAX Formulas

The analysis utilizes optimized **DAX (Data Analysis Expressions)** measures organized in a Dedicated Measure Table (`_Measures`).

### Core Financial & Operational Measures

```dax
// 1. Total Revenue
Total Revenue = SUM('Fact_Sales'[SalesAmount])

// 2. Total Cost of Goods Sold
Total COGS = SUM('Fact_Sales'[TotalCost])

// 3. Gross Profit
Gross Profit = [Total Revenue] - [Total COGS]

// 4. Gross Margin %
Gross Margin % = 
DIVIDE(
    [Gross Profit], 
    [Total Revenue], 
    0
)

// 5. Total Units Sold
Total Quantity = SUM('Fact_Sales'[OrderQuantity])

// 6. Average Order Value (AOV)
Average Order Value = 
DIVIDE(
    [Total Revenue], 
    DISTINCTCOUNT('Fact_Sales'[SalesID]), 
    0
)
```

### Time Intelligence Measures (YTD, YoY Growth)

```dax
// 7. Revenue Year-to-Date (YTD)
Revenue YTD = 
TOTALYTD(
    [Total Revenue], 
    'Dim_Date'[Date]
)

// 8. Prior Year Revenue (SPLY)
Revenue PY = 
CALCULATE(
    [Total Revenue], 
    SAMEPERIODLASTYEAR('Dim_Date'[Date])
)

// 9. Year-over-Year (YoY) Growth %
YoY Revenue Growth % = 
DIVIDE(
    [Total Revenue] - [Revenue PY], 
    [Revenue PY], 
    0
)
```

---

## 5. Report Visualizations & Page Structure

The Power BI report layout consists of four interactive analytical dashboards:

### Page 1: Executive KPI Summary
- **Card Visuals**: Total Revenue, Gross Margin %, YTD Revenue, YoY Growth %.
- **Line & Clustered Column Chart**: Monthly Revenue Trend vs. Prior Year Revenue.
- **Donut Chart**: Revenue Breakdown by Customer Segment.
- **Slicers**: Date Range Slider, Geographic Region, Product Category.

### Page 2: Product & Category Analysis
- **Matrix Visual**: Category & Sub-Category Hierarchy showing Units Sold, Margin %, and Discount impact.
- **Bar Chart**: Top 10 Performing Products by Net Sales.
- **Scatter Plot**: Unit Cost vs. Sales Volume (identifying high-margin drivers).

### Page 3: Regional Performance & Geography
- **Map Visual**: Regional Sales distribution across territories.
- **Decomposition Tree**: Drill-down analysis from Region -> Sales Rep -> Customer Segment.
- **Table Visual**: Country-level breakdown of SLA fulfillment and delivery performance.

### Page 4: Customer Insights & Cohort Analysis
- **Stacked Bar Chart**: Active vs. At-Risk Customers by RFM (Recency, Frequency, Monetary) Score.
- **Key Influencers Visual**: Drivers impacting high-value customer retention.

---

## 6. Data Governance, Security & Performance Optimization

1. **Row-Level Security (RLS)**:
   - Defined roles (`Regional_Manager`, `Executive_Viewer`, `Sales_Rep`) restricting data access based on `USERPRINCIPALNAME()`.
2. **Performance Tuning**:
   - High-cardinality columns (e.g., raw timestamps) were split or removed to optimize VertiPaq engine memory usage.
   - Dual/Import storage modes optimized for query responsiveness.
3. **Data Refresh & ETL**:
   - Power Query M transformations applied at the source to clean NULL values, cast strict data types, and enforce referential integrity.

---

## 7. How to Use This Report

1. **Opening the Model**: Open `Report.pbix` using Power BI Desktop (2024 or later).
2. **Updating Data Sources**: Navigate to `Transform Data` -> `Data Source Settings` to re-point SQL Server or Web API endpoints.
3. **Exploring Insights**: Use page-level slicers and cross-filtering across visuals to perform root-cause analysis.
4. **Publishing**: Publish to Power BI Service workspace and configure Gateway schedule refresh (Daily/Hourly).

---
*Report Generated Automatically for Power BI Project Documentation.*
