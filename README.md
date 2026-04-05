# E-Wallet_Transactional_Analysis_by_Python :Data Wrangling & Optimization

## 📑 Table of Contents  
1. [📌 Background & Overview](#-background--overview)  
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)
4. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)

---

## 📌 Background & Overview

### Background

This project aims to evaluate the payment and transaction health of an E-wallet ecosystem. By analyzing product payment volumes and raw transaction logs, this project uncovers actionable insights into business performance.

### Core Business Questions:
* **Business Performance Tracking:** Which products are driving the highest payment volumes? Which teams and categories have been underperforming since Q2.2023 and need strategic intervention?

* **Transaction Behavior & Labeling:** How are millions of raw transaction logs classified into meaningful business actions (e.g., Bank Transfers, Top-ups, Withdrawals, Split Bills)? What is the volume and user distribution across these types?

* **Risk & Logic Validation:** Are there data anomalies (e.g., a single product violating the "one team" rule)? Which sources are responsible for the highest refund contributions?

---

## 📂 Dataset Description & Data Structure

## 📂 Data Source

* **Source:** Provided internal E-wallet system datasets.

* **Size:** 3 datasets containing product metadata, aggregated monthly volumes, and raw transaction logs. Over **1.3 million rows** of transaction data"

* **Format:** `.csv`

## 📂 Data Structure
**Table 1:** `product.csv` **(Product Information)** 

<img width="339" height="100" alt="image" src="https://github.com/user-attachments/assets/4a0d50a2-77b6-4f9b-910e-a3f1ec15539b" />

| Column Name | Data Type |
| :--- | :--- | 
| `product_id` | INT | 
| `team` | OBJECT |
| `category` | OBJECT |

**Table 2: `payment_report.csv` (Monthly Payment Volume)** 

<img width="573" height="110" alt="image" src="https://github.com/user-attachments/assets/dc218f43-8466-465a-b7c8-3d0fa0b40d89" />


| Column Name | Data Type | 
| :--- | :--- | 
| `report_month` | OBJECT  |
| `payment_group` | OBJECT  | 
| `product_id` | INT64 | 
| `source_id` | INT64 | 
| `volume` | INT64 | 

**Table 3: `transactions.csv` (Transactions Information)**

<img width="1037" height="121" alt="image" src="https://github.com/user-attachments/assets/5c80f286-38af-445a-8bf4-848e37ae2fe8" />

Column Name | Data Type | 
| :--- | :--- | 
| `transaction_id` | INT64 | 
| `merchant_id` | INT64 | 
| `volume` | INT64 | 
| `transType` | INT64 |
| `transStatus` | INT64 | 
| `sender_id` | FLOAT64 | 
| `receiver_id` | FLOAT64 | 
| `extra_info` | OBJECT |
| `timeStamp` | INT64 |  

---

## ⚒️ Main Process

### 1️⃣ Data Cleaning & Preprocessing
Before diving into the analysis, I processed the raw datasets to ensure data integrity and accuracy for downstream calculations.

👉🏻 *Transformations applied:*
* **Data Type Conversion:** Converted `report_month` in `payment_report.csv` from an object (string) to `datetime64[ns]` to enable time-series filtering (e.g., extracting data >= Q2.2023).
* **Handling Duplicates:** Identified and removed 28 duplicate records in the `transactions.csv` dataset to prevent inflated financial metrics.
* **Handling Missing Values:** * `category` & `team_own`: 22 missing rows (~2%). Kept as-is since these product IDs didn't exist in the master table.
    * `sender_id` & `receiver_id`: Nulls were retained because certain transaction types inherently lack this info.

---

### 2️⃣ Exploratory Data Analysis (EDA)
To understand the underlying distribution and patterns of the data, I utilized basic Pandas functions.

👉🏻 *Key Steps:*
* Checked the shape, schema, and memory usage using `df.info()` and `df.shape`.
* Deployed **`ydata-profiling`** to generate a comprehensive HTML report, quickly surfacing distributions, correlations, and zero-variance columns across the 1.3M+ transaction records.

---

### 3️⃣ Python Analysis

## Task 1: Product Performance & Master Data Integrity
**Business rule validation:** "1 `product_id` is only owned by 1 `team`."
👉🏻 *Code Purpose:* Merged payment and product data (`how='left'`). Grouped by `product_id` using `.nunique()` on teams to find violations, and calculated the top 3 highest-volume products.

**Results:**
* **Integrity Check:** `0` products violated the rule. Master data is perfectly clean.
* **Top 3 Products:**
  * ID 1976: 61,797,583,647 (Volume)
  * ID 429: 14,667,676,567
  * ID 372: 13,713,658,515

## Task 2: Underperforming Units & Refund Risk Alert
👉🏻 *Code Purpose:* Filtered data from `2023-04-01` onwards. Aggregated volume by `team` and `category` to isolate the lowest performers. For refunds, filtered `payment_group == 'refund'` to pinpoint the highest contributing `source_id`.

**Results:**
* **Lowest Team:** Team **APS** (Volume: 51,141,753).
* **Lowest Category:** **PXXXXXE** within Team APS (Volume: 25,232,438).
* **Refund Alert:** `source_id` **38** generated an abnormally high refund volume of **36.5 Billion**, requiring immediate technical review.

## Task 3: Transaction Labeling & Profiling
Raw transactions lack business context. 

 **Code Purpose:** Created a custom function (`classify_trac`) using `if/elif` logic to label 1.3M rows based on `transType` and `merchant_id` combinations (e.g., Code 2 + Merchant 2270 = Top Up). Excluded 'Invalid' transactions and aggregated metrics using `.agg()`.

**Results:**
| transaction_type | num_transactions | total_volume | unique_senders | unique_receivers |
| :--- | :--- | :--- | :--- | :--- |
| Payment Transaction | 398,665 | 71,850,608,441 | 139,583 | 113,298 |
| Top Up Money Transaction | 290,498 | 108,605,618,829 | 110,409 | 110,409 |
| Transfer Money Transaction | 341,173 | 37,032,880,492 | 39,021 | 34,585 |
| Bank Transfer Transaction | 37,879 | 50,605,806,190 | 23,156 | 9,271 |
| Withdraw Money Transaction| 33,725 | 23,418,181,420 | 24,814 | 24,814 |
| Split Bill Transaction | 1,376 | 4,901,464 | 1,323 | 572 |


## 🔎 Final Conclusion & Recommendations  

**Key Takeaways:**

* **Risk Alert:** While dataset is perfectly clean, `source_id` 38 accounts for an abnormal 36.5B in refunds.
* **Performance Gap:** Product `1976` heavily drives revenue (61.7B), while Team `APS` (category `PXXXXXE`) severely underperforms.

🚀 **Recommendations:** 
* ✔️ **Audit Source 38:** Immediately investigate this gateway for technical failures or fraud to stop revenue leakage.
* ✔️ **Optimize Team APS:** Revamp marketing for category `PXXXXXE` or consider retiring it to save operational resources.
