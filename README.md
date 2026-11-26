# Customer Shopping Behavior Analysis

**End-to-end analysis of 3,900 retail customers using Python, PostgreSQL and Power BI.**

![Python](https://img.shields.io/badge/Python-pandas-blue) ![SQL](https://img.shields.io/badge/SQL-PostgreSQL-336791) ![Power BI](https://img.shields.io/badge/Dashboard-Power%20BI-F2C811)

---

##  Table of Contents
1. [Business Problem](#-business-problem)
2. [About the Data](#-about-the-data)
3. [Approach and Tools](#-approach-and-tools)
4. [Analysis Results](#-analysis-results)
5. [Power BI Dashboard](#-power-bi-dashboard)
6. [Recommendations](#-recommendations)
7. [Limitations](#-limitations)
8. [Repository Structure](#-repository-structure)

---

## Business Problem

A retail business sells clothing, accessories, footwear and outerwear, and wants to understand **who its customers are and what drives their spending**. Leadership needs answers to questions such as:

- Which customer groups (gender, age, subscribers vs. non-subscribers) generate the most revenue?
- Do discounts and subscriptions actually lead to higher spending?
- Which products and categories perform best by volume, rating and discount reliance?
- How loyal is the customer base, and are repeat buyers more likely to subscribe?
- Does the shipping option a customer chooses relate to how much they spend?

**Goal:** turn raw purchase data into insights that guide marketing, pricing and loyalty decisions.

---

##  About the Data

| Item | Detail |
|---|---|
| File | `Customer_Shopping_Behavior.csv` |
| Size | 3,900 rows Ã— 18 columns (one row per customer, unique `Customer ID`) |
| Geography | 50 US states |
| Categories | Clothing, Accessories, Footwear, Outerwear |

**Columns**

| Group | Fields |
|---|---|
| Demographics | Customer ID, Age (18â€“70), Gender, Location, Subscription Status |
| Purchase details | Item Purchased, Category, Purchase Amount (USD, $20â€“$100), Size, Color, Season |
| Behaviour | Discount Applied, Promo Code Used, Previous Purchases (1â€“50), Frequency of Purchases, Review Rating (2.5â€“5.0), Shipping Type, Payment Method |

**Data quality checks and cleaning**

| Issue | Action |
|---|---|
| 37 missing values in `Review Rating` (about 1%) | Imputed with the **median rating of each product category**, so one category's ratings do not skew another's |
| Inconsistent column names | Converted to `snake_case` (e.g. `Purchase Amount (USD)` â†’ `purchase_amount`) |
| `Discount Applied` and `Promo Code Used` identical in all 3,900 rows | Confirmed redundant and **dropped `promo_code_used`** |
| No age grouping | Created `age_group` with quartile binning (`pd.qcut`): Young Adult 18â€“31, Adult 32â€“44, Middle-aged 45â€“57, Senior 58â€“70 |
| Purchase frequency stored as text | Created `purchase_frequency_days` (e.g. Weekly = 7, Monthly = 30, Annually = 365) |

---

## Approach and Tools

1. **Python (pandas):** loading, profiling, cleaning and feature engineering (`Customer_Shopping_Behavior-Analysis.ipynb`).
2. **PostgreSQL:** the cleaned dataframe was loaded into a database through SQLAlchemy, then business questions were answered with SQL using aggregations, CTEs, subqueries, `CASE` and window functions (`Customer_Behavior_SQL_QUERIES.sql`).
3. **Power BI:** an interactive dashboard connected to the PostgreSQL table (`Customer_Behavior_Dashboard.pbix`).

---

##  Analysis Results

**Overall:** total revenue of **$233,081**, average purchase of **$59.76**, average review rating of about **3.75 / 5**.

### 1. Revenue by gender

| Gender | Customers | Total revenue | Share of revenue | Avg purchase |
|---|---|---|---|---|
| Male | 2,652 | $157,890 | 67.7% | $59.54 |
| Female | 1,248 | $75,191 | 32.3% | $60.25 |

![Revenue by gender](images/revenue_by_gender.png)

**Insight:** men generate about two-thirds of revenue, but only because they are about two-thirds of the customer base. **Average spend per purchase is nearly identical** ($59.54 vs. $60.25).

### 2. Revenue by category

| Category | Revenue | Share |
|---|---|---|
| Clothing | $104,264 | 44.7% |
| Accessories | $74,200 | 31.8% |
| Footwear | $36,093 | 15.5% |
| Outerwear | $18,524 | 7.9% |

![Revenue by category](images/revenue_by_category.png)

**Insight:** Clothing and Accessories together bring in about 76% of revenue. Outerwear is the smallest category, with a lower average purchase ($57.17 vs. about $60 elsewhere).

### 3. Revenue by age group

| Age group | Revenue | Share | Avg purchase |
|---|---|---|---|
| Young Adult (18â€“31) | $62,143 | 26.7% | $60.45 |
| Middle-aged (45â€“57) | $59,197 | 25.4% | $60.04 |
| Adult (32â€“44) | $55,978 | 24.0% | $59.42 |
| Senior (58â€“70) | $55,763 | 23.9% | $59.07 |

![Revenue by age group](images/revenue_by_age_group.png)

**Insight:** revenue is spread fairly evenly across age groups. Young Adults lead, but the gap between the top and bottom group is under 3 percentage points.

### 4. Subscribers vs. non-subscribers

| Status | Customers | Avg spend | Total revenue |
|---|---|---|---|
| Non-subscribers | 2,847 (73%) | $59.87 | $170,436 |
| Subscribers | 1,053 (27%) | $59.49 | $62,645 |

![Subscription split](images/subscription_split.png)

**Insight:** subscribers do **not** spend more per purchase. Non-subscribers bring in more total revenue only because there are far more of them.

**Notable pattern:** in this dataset **none of the 1,248 female customers are subscribers**, while 39.7% of male customers are. Subscription is effectively a male-only feature in this data (see Limitations).

### 5. Customer loyalty segments

Segments are based on `previous_purchases`: **New** = 1, **Returning** = 2â€“10, **Loyal** = 11 or more.

| Segment | Customers | Share |
|---|---|---|
| Loyal | 3,116 | 79.9% |
| Returning | 701 | 18.0% |
| New | 83 | 2.1% |

![Customer segments](images/customer_segments.png)

**Insight:** the customer base is overwhelmingly repeat buyers. Only 2% are first-time purchasers, which points to strong retention but a weak flow of new customers.

### 6. Are repeat buyers more likely to subscribe?

Among customers with more than 5 previous purchases: **958 of 3,476 (27.6%)** subscribe, compared with **27.0%** across all customers.

**Insight:** repeat buying has essentially **no link to subscribing**. Frequent buyers are not being converted into subscribers.

### 7. Discounts

- **43%** of purchases (1,677) used
