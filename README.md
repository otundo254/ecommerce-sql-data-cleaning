# ecommerce-sql-data-cleaning
SQL-based data cleaning pipeline for e-commerce transaction data, including handling NULL values, duplicates, and data standardization.
# 📊 E-commerce Sales Data Cleaning & Analysis (SQL + Power BI)

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

## Data Import (PostgreSQL)

The dataset was imported into PostgreSQL using the following command:

```sql
\COPY online_retail_raw
FROM 'C:/data/online_retail.csv'
WITH (FORMAT csv, HEADER true);

A raw table (online_retail_raw) was used to store the original data without modification.

---

## Data Cleaning Steps

A layered approach was used:

- online_retail_raw → raw data
- clean_retail_data_v1 → handled NULL values
- clean_retail_data_v2 → removed duplicates
- clean_retail_data_v3 → formatted and analysis-ready data

---

## Step 1: Handling NULL Values

NULL values were handled using COALESCE().

Key fixes:
- Missing CustomerID → replaced with '0'
- Missing Description → replaced with 'UNKNOWN PRODUCT'

Example:
COALESCE(customerid, '0')
COALESCE(description, 'UNKNOWN PRODUCT')

---

## Step 2: Removing Duplicates

Duplicates were removed using ROW_NUMBER().

Logic used:
ROW_NUMBER() OVER (
    PARTITION BY invoiceno, stockcode, description, quantity, invoicedate, unitprice, customerid, country
)

Only the first row was kept:
WHERE row_num = 1

---

## Step 3: Standardization & Formatting

Data was cleaned and standardized:

- Converted text to uppercase using UPPER()
- Removed extra spaces using TRIM()
- Converted dates using TO_TIMESTAMP()

Example:
TRIM(UPPER(description))
TO_TIMESTAMP(invoicedate, 'DD/MM/YYYY HH24:MI')

---

## Derived Columns

New columns created:

- transaction_type → SALE or RETURN
- total_amount → quantity × unit price

Logic:
CASE 
    WHEN quantity < 0 THEN 'RETURN'
    ELSE 'SALE'
END

quantity * unitprice

---

## Data Challenges

The dataset included:
- Missing customer IDs
- Duplicate rows
- Inconsistent text formatting
- Negative quantities (returns)

These were handled using SQL cleaning techniques.

---

