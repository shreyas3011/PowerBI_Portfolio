# Olist E-Commerce Data Analysis | PostgreSQL & Power BI

## 1. Project Overview

This project focuses on analyzing Olist e-commerce business data using **PostgreSQL and Microsoft Power BI**. The objective is to transform raw CSV data into a structured relational database, develop SQL-based data preparation logic, and build an interactive Power BI dashboard to analyze sales performance, customer behavior, payments, delivery operations, product reviews, and seller performance.

The project follows an end-to-end data analytics workflow, starting with raw data ingestion and database development and ending with business-focused reporting and interactive analysis.

## 2. Technology Stack

| Technology | Purpose |
|---|---|
| PostgreSQL | Relational database management |
| SQL | Data transformation, joins, views, and indexing |
| Microsoft Power BI | Data visualization and reporting |
| DAX | Measures, KPIs, and time-based calculations |
| CSV | Source data format |

## 3. Project Workflow

1. Collected the project data in CSV format.
2. Created a PostgreSQL database named `olist_db`.
3. Designed the relational database structure.
4. Created 9 tables with primary key and foreign key relationships.
5. Imported CSV data into the respective database tables.
6. Created indexes on selected columns to support query performance.
7. Developed one SQL view containing selected important columns for reporting.
8. Connected the PostgreSQL data to Power BI.
9. Developed DAX calculations for business KPIs and performance analysis.
10. Built dashboard pages for different business areas, including drill-through analysis.

## 4. Database Architecture

**Database:** `olist_db`

The database contains 9 tables organized using primary key and foreign key relationships.

| Database Component | Implementation |
|---|---|
| Database | `olist_db` |
| Total tables | 9 |
| Primary keys | Created |
| Foreign keys | Created |
| Indexes | Created on selected columns |
| SQL views | 1 |
| Data source | CSV files |

The relational structure helps organize the e-commerce data and establish connections between different business entities.

### SQL Implementation

- Database creation and table design.
- Primary key and foreign key implementation.
- CSV data loading.
- Index creation on selected columns.
- SQL view creation to select important reporting columns.

## 5. Power BI Dashboard and DAX Measures

The Power BI report uses DAX to calculate business metrics, performance indicators, and analytical measures.

The following measures are based on the DAX definitions provided for this project.

### A. Base Business Measures

These measures establish the core metrics used throughout the dashboard.

| Measure | Business Purpose |
|---|---|
| Revenue | Total product sales revenue |
| Freight | Total freight charges |
| GMV | Revenue including freight |
| Orders | Distinct count of orders |
| Customers | Distinct count of unique customers |
| Items Sold | Total number of fact table rows |
| AOV | Average revenue per order |
| Items per Order | Average items per order |
| Avg Rating | Average customer review score |
| Delivered Orders | Count of delivered orders |
| Late Orders | Count of delivered orders marked late |
| On-time % | Percentage of delivered orders that were not late |

**DAX:**

```dax
Revenue := SUM ( 'public bi_fact_sales'[price] )

Freight := SUM ( 'public bi_fact_sales'[freight_value] )

GMV := [Revenue] + [Freight]

Orders := DISTINCTCOUNT ( 'public bi_fact_sales'[order_id] )

Customers := DISTINCTCOUNT ( 'public bi_fact_sales'[customer_unique_id] )

Items Sold := COUNTROWS ( 'public bi_fact_sales' )

AOV := DIVIDE ( [Revenue], [Orders] )

Items per Order := DIVIDE ( [Items Sold], [Orders] )

Avg Rating := AVERAGE ( 'public bi_fact_sales'[review_score] )

Delivered Orders :=
CALCULATE (
    [Orders],
    'public bi_fact_sales'[order_status] = "delivered"
)

Late Orders :=
CALCULATE (
    [Orders],
    'public bi_fact_sales'[order_status] = "delivered",
    'public bi_fact_sales'[is_late] = 1
)

On-time % :=
DIVIDE (
    [Delivered Orders] - [Late Orders],
    [Delivered Orders]
)
```

### B. Time Intelligence Measures

These measures support historical comparisons, year-to-date performance, and rolling revenue analysis.

| Measure | Business Purpose |
|---|---|
| Revenue YTD | Revenue accumulated from the beginning of the year |
| Revenue LY | Revenue for the equivalent period in the previous year |
| YoY % | Year-over-year revenue growth percentage |
| Rolling 30D Revenue | Revenue over the latest 30-day period |

**DAX:**

```dax
Revenue YTD :=
TOTALYTD ( [Revenue], DimDate[Date] )

Revenue LY :=
CALCULATE (
    [Revenue],
    SAMEPERIODLASTYEAR ( DimDate[Date] )
)

YoY % :=
DIVIDE (
    [Revenue] - [Revenue LY],
    [Revenue LY]
)

Rolling 30D Revenue :=
CALCULATE (
    [Revenue],
    DATESINPERIOD (
        DimDate[Date],
        MAX ( DimDate[Date] ),
        -30,
        DAY
    )
)
```

### C. Review Sentiment Measures

These measures analyze customer satisfaction by classifying review scores into positive and negative sentiment percentages.

| Measure | Business Purpose |
|---|---|
| Positive Reviews % | Percentage of reviews with scores of 4 or 5 |
| Negative Reviews % | Percentage of reviews with scores of 1 or 2 |

**DAX:**

```dax
Positive Reviews % :=
DIVIDE (
    CALCULATE (
        COUNTROWS ( 'public bi_fact_sales' ),
        'public bi_fact_sales'[review_score] >= 4
    ),
    CALCULATE (
        COUNTROWS ( 'public bi_fact_sales' ),
        NOT ISBLANK ( 'public bi_fact_sales'[review_score] )
    )
)

Negative Reviews % :=
DIVIDE (
    CALCULATE (
        COUNTROWS ( 'public bi_fact_sales' ),
        'public bi_fact_sales'[review_score] <= 2
    ),
    CALCULATE (
        COUNTROWS ( 'public bi_fact_sales' ),
        NOT ISBLANK ( 'public bi_fact_sales'[review_score] )
    )
)
```

### D. Payment Measures

These measures evaluate payment value and installment behavior.

| Measure | Business Purpose |
|---|---|
| Payment Value | Total payment value |
| Avg Installments | Average number of payment installments |

**DAX:**

```dax
Payment Value :=
SUM ( 'public bi_fact_sales'[payment_value] )

Avg Installments :=
AVERAGE ( 'public bi_fact_sales'[payment_installments] )
```

### E. Delivery and Logistics Measures — Page 5

These measures evaluate delivery duration and late delivery performance.

| Measure | Business Purpose |
|---|---|
| Avg Delivery Days | Average delivery duration |
| Late % | Percentage of delivered orders that were late |

**DAX:**

```dax
Avg Delivery Days :=
AVERAGE ( 'public bi_fact_sales'[delivery_days] )

Late % :=
DIVIDE ( [Late Orders], [Delivered Orders] )
```

### F. Review Analysis — Page 6

The Review Bucket calculation classifies review scores into four categories.

| Review Score | Category |
|---|---|
| Blank | No Review |
| 4–5 | Positive |
| 3 | Neutral |
| 1–2 | Negative |

**DAX:**

```dax
Review Bucket =
SWITCH (
    TRUE(),
    ISBLANK ( 'public bi_fact_sales'[review_score] ), "No Review",
    'public bi_fact_sales'[review_score] >= 4, "Positive (4-5)",
    'public bi_fact_sales'[review_score] = 3, "Neutral (3)",
    "Negative (1-2)"
)
```

*Note: This expression is a calculated column rather than a measure because it evaluates the review score for individual rows.*

### G. Seller Performance — Page 8

These measures support seller-level performance analysis.

| Measure | Business Purpose |
|---|---|
| Sellers | Distinct count of sellers |
| Revenue per Seller | Average revenue per seller |

**DAX:**

```dax
Sellers :=
DISTINCTCOUNT ( 'public bi_fact_sales'[seller_id] )

Revenue per Seller :=
DIVIDE ( [Revenue], [Sellers] )
```

### H. Drill-through Analysis — Page 9

The Drill Title measure dynamically displays the selected product category in the drill-through page title.

**DAX:**

```dax
Drill Title =
"Drillthrough: " &
COALESCE (
    SELECTEDVALUE ( 'public bi_fact_sales'[category] ),
    "All"
)
```

## 6. Key KPIs Covered in the Dashboard

The DAX measures support analysis across the following KPI areas:

| Business Area | KPIs |
|---|---|
| Sales Performance | Revenue, Freight, GMV, AOV |
| Order Performance | Orders, Items Sold, Items per Order |
| Customer Analysis | Customers |
| Time-Based Performance | Revenue YTD, Revenue LY, YoY %, Rolling 30D Revenue |
| Delivery Performance | Delivered Orders, Late Orders, On-time %, Late %, Avg Delivery Days |
| Customer Satisfaction | Avg Rating, Positive Reviews %, Negative Reviews % |
| Payment Analysis | Payment Value, Avg Installments |
| Seller Performance | Sellers, Revenue per Seller |

## 7. Business Analysis and Insights

The dashboard's measures enable analysis of several important business questions:

**Sales and Revenue**
- How much revenue and GMV does the business generate?
- What is the average order value?
- How does revenue change over time?
- How does current revenue compare with the previous year?

**Customer and Order Performance**
- How many unique customers and orders are recorded?
- How many items are sold?
- What is the average number of items per order?

**Delivery and Logistics**
- What is the average delivery duration?
- What percentage of delivered orders are late?
- What is the on-time delivery percentage?

**Customer Satisfaction**
- What is the average review score?
- What percentage of reviews are positive or negative?
- How are customer reviews distributed across sentiment categories?

**Payment Analysis**
- What is the total payment value?
- What is the average installment count?

**Seller Performance**
- How many distinct sellers are present?
- What is the revenue per seller?

These are the analytical questions supported by the provided DAX measures. Actual findings, such as revenue growth rates, the highest-performing category, or the percentage of late deliveries, should be added only after validating the dashboard results.

## 8. Project Structure

```text
Olist-Ecommerce-Data-Analysis/
│
├── Dataset/
│   └── CSV source files
│
├── SQL/
│   ├── Database_Creation.sql
│   ├── Table_Creation.sql
│   ├── Relationships.sql
│   ├── Indexes.sql
│   └── Views.sql
│
├── PowerBI/
│   ├── Olist_Dashboard.pbix
│   └── DAX Measures.txt
│
├── Screenshots/
│   └── Dashboard screenshots
│
└── README.md
```

## 9. Skills Demonstrated

- **SQL and Database Management:** Database creation, relational table design, primary keys, foreign keys, indexing, and views.
- **Data Preparation:** CSV data ingestion and preparation for reporting.
- **Power BI:** Connecting PostgreSQL data and developing analytical dashboards.
- **DAX:** Aggregation measures, filter context, time intelligence, conditional calculations, and dynamic titles.
- **Business Analysis:** Sales, customer, delivery, payment, review, and seller performance analysis.
- **Data Visualization:** KPI reporting and interactive drill-through analysis.

## 10. Conclusion

This project demonstrates an end-to-end e-commerce data analytics workflow using PostgreSQL, SQL, Power BI, and DAX. It combines relational database development with business-focused reporting to analyze sales, customer behavior, payment activity, logistics, reviews, and seller performance.

The project showcases practical skills in database design, SQL querying, data modeling, DAX calculations, KPI development, and business intelligence reporting.
