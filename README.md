# ecommerce-sql-data-cleaning
SQL-based data cleaning pipeline for e-commerce transaction data, including handling NULL values, duplicates, and data standardization.
#  E-commerce Sales Data Cleaning & Analysis (SQL + Power BI)

## 📌 Project Overview

This project focuses on cleaning and preparing e-commerce transaction data using SQL, and later visualizing insights using Power BI.

The goal is to simulate a real-world data analyst workflow:
- Import raw data into a database
- Clean and transform the data using SQL
- Prepare it for analysis and visualization

---

## Data Source

- Dataset: Online Retail Dataset (UCI)
- Format: CSV (converted from Excel)
- Records: 541,909 rows

The dataset contains transaction-level data including invoices, products, customers, and countries.

---

## Tools Used

- PostgreSQL (data storage & SQL processing)
- SQL (data cleaning & transformation)
- Power BI (data visualization)
  
## Data Import (PostgreSQL)

The dataset was imported into PostgreSQL using:

```sql
\COPY online_retail_raw
FROM 'C:/data/online_retail.csv'
WITH (FORMAT csv, HEADER true);
```

A raw table (online_retail_raw) was used to store the original data without modification.

## Data Cleaning Steps

A layered approach was used:

online_retail_raw → raw data
clean_retail_data_v1 → handled NULL values
clean_retail_data_v2 → removed duplicates
clean_retail_data_v3 → formatted and analysis-ready data
### Step 1: Handling NULL Values

NULL values were handled using COALESCE().

Key fixes:

Missing CustomerID → replaced with '0'
Missing Description → replaced with 'UNKNOWN PRODUCT'

Example:

```sql
COALESCE(customerid, '0') AS customerid,
COALESCE(description, 'UNKNOWN PRODUCT') AS description
```
### Step 2: Removing Duplicates

Duplicates were removed using ROW_NUMBER().

```sql
ROW_NUMBER() OVER (
    PARTITION BY invoiceno, stockcode, description, quantity, invoicedate, unitprice, customerid, country
) AS row_num
```

Only the first row was kept:

```sql
WHERE row_num = 1
```
### Step 3: Standardization & Formatting

Data was cleaned and standardized:

Converted text to uppercase using UPPER()
Removed extra spaces using TRIM()
Converted dates using TO_TIMESTAMP()

Example:

```sql
TRIM(UPPER(description)) AS description,
TO_TIMESTAMP(invoicedate, 'DD/MM/YYYY HH24:MI') AS invoice_date
```
## Derived Columns

New columns created:

transaction_type → SALE or RETURN
total_amount → quantity × unit price
```sql
CASE 
    WHEN quantity < 0 THEN 'RETURN'
    ELSE 'SALE'
END AS transaction_type,

quantity * unitprice AS total_amount
```

## 📊 Data Analysis (SQL)

Key business metrics were calculated using SQL queries.

### Core KPIs


Total Revenue, Total Orders, Total customers & Average Order Value

```sql
SELECT
    SUM(total_amount) AS total_revenue,
    COUNT(DISTINCT invoiceno) AS total_orders,
    COUNT(DISTINCT customerid) AS total_customers,
    ROUND(SUM(total_amount) / COUNT(DISTINCT invoiceno), 2) AS avg_order_value
FROM clean_retail_data_v3
WHERE transaction_type = 'SALE';
```
### Revenue over Time
```sql
SELECT
    DATE_TRUNC('month', invoice_date) AS month,
    SUM(total_amount) AS revenue
FROM clean_retail_data_v3
WHERE transaction_type = 'SALE'
GROUP BY month
ORDER BY month;
```
### Top products
```sql
SELECT
    description,
    SUM(quantity) AS total_units_sold,
    SUM(total_amount) AS revenue
FROM clean_retail_data_v3
WHERE transaction_type = 'SALE'
GROUP BY description
ORDER BY revenue DESC
LIMIT 10;
```
### Sales by country
```sql
SELECT
    country,
    SUM(total_amount) AS revenue
FROM clean_retail_data_v3
WHERE transaction_type = 'SALE'
GROUP BY country
ORDER BY revenue DESC;
```
### Top customers
```sql
SELECT
    customerid,
    COUNT(DISTINCT invoiceno) AS total_orders,
    SUM(total_amount) AS total_spent
FROM clean_retail_data_v3
WHERE transaction_type = 'SALE'
GROUP BY customerid
ORDER BY total_spent DESC
LIMIT 10;
```
### Return Analysis
Shows Revenue that was lost to Returns
```sql
SELECT
    transaction_type,
    COUNT(*) AS total_transactions,
    SUM(total_amount) AS total_value
FROM clean_retail_data_v3
GROUP BY transaction_type;
```
##  Power BI Dashboard

The cleaned dataset (`clean_retail_data_v3`) was connected to Power BI to build an interactive dashboard for business analysis.

---

##  Data Connection

Power BI was connected directly to PostgreSQL using the PostgreSQL connector.

- Server: `localhost`
- Database: `portfolio_projects`
- Table/View used: `public.clean_retail_data_v3`

---

##  Dashboard Components

### 🔹 KPI Cards

The following key metrics were created using DAX:

- Total Revenue (includes Returns)
- Net Revenue (Sales only)
- Total Orders
- Total Customers
- Average Order Value
- Return Rate (%)

Example:

Total Revenue =
CALCULATE(
    SUM(clean_retail_data_v3[total_amount]),
    clean_retail_data_v3[transaction_type] = "SALE"
)

---

### 🔹 Revenue Trend

A line chart was used to visualize revenue over time:

- Axis: invoice_date
- Values: Total Revenue

This helps identify trends and seasonal patterns.

---

### 🔹 Top Products

A bar chart showing the top 10 products by revenue:

- Axis: description
- Values: Total Revenue
- Filter: Top 10 by revenue

---

### 🔹 Top Customers

A bar chart showing the highest revenue-generating customers:

- Axis: customerid
- Values: Customer Revenue
- Filter: Top 10 customers
- Note: Unknown customers (`customerid = '0'`) 

---

### 🔹 Returns Analysis

Returns were analyzed using a KPI metric:

- Return Rate (%) = Returns ÷ Sales

Due to the small proportion of returns, a KPI card was used instead of a pie chart for better clarity.

---

##  Filters (Slicers)

The dashboard includes interactive filters:

- Date slicer (Month/Year or relative date)
- Transaction type (SALE / RETURN)

These allow users to dynamically explore the data.

---

##  Key Insights

- Top customers contribute a significant portion of total revenue, indicating revenue concentration. Dotcom Postage was the leading product with over 200k sales.
- Return rate is approximately 8.4% of the total revenue, suggesting low product return levels
- Revenue shows steady monthly performance with identifiable peaks in the month of september to the begining of Noverber 2011.

---

##  Dashboard Preview

<img width="872" height="421" alt="image" src="https://github.com/user-attachments/assets/8fc99b3a-c55a-4e7b-9a81-ecaa5eca76fd" />

## Data Challenges

The dataset included:

Missing customer IDs
Duplicate rows
Inconsistent text formatting
Negative quantities (returns)

These were handled using SQL cleaning techniques.


