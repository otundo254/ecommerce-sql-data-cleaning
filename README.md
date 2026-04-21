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

##  Data Source

- Dataset: Online Retail Dataset (UCI)
- Format: CSV (converted from Excel)
- Records: 541,909 rows

The dataset contains transaction-level data including invoices, products, customers, and countries.

---

## 🛠 Data Import (PostgreSQL)

The dataset was imported into PostgreSQL using:

```sql
\COPY online_retail_raw
FROM 'C:/data/online_retail.csv'
WITH (FORMAT csv, HEADER true);
