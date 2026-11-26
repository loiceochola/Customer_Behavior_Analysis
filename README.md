# Customer Shopping Behavior Analysis

**End-to-end analysis of 3,900 retail customers using Python, PostgreSQL and Power BI.**

![Python](https://img.shields.io/badge/Python-pandas-blue) ![SQL](https://img.shields.io/badge/SQL-PostgreSQL-336791) ![Power BI](https://img.shields.io/badge/Dashboard-Power%20BI-F2C811)

---

## 📌 Table of Contents
1. [Business Problem](#-business-problem)
2. [About the Data](#-about-the-data)
3. [Approach and Tools](#-approach-and-tools)
4. [Analysis Results](#-analysis-results)
5. [Power BI Dashboard](#-power-bi-dashboard)
6. [Recommendations](#-recommendations)
7. [Limitations](#-limitations)
8. [Repository Structure](#-repository-structure)

---

## 🎯 Business Problem

A retail business sells clothing, accessories, footwear and outerwear, and wants to understand **who its customers are and what drives their spending**. Leadership needs answers to questions such as:

- Which customer groups (gender, age, subscribers vs. non-subscribers) generate the most revenue?
- Do discounts and subscriptions actually lead to higher spending?
- Which products and categories perform best by volume, rating and discount reliance?
- How loyal is the customer base, and are repeat buyers more likely to subscribe?
- Does the shipping option a customer chooses relate to how much they spend?

**Goal:** turn raw purchase data into insights that guide marketing, pricing and loyalty decisions.

---

## 🗂 About the Data

| Item | Detail |
|---|---|
| File | `Customer_Shopping_Behavior.csv` |
| Size | 3,900 rows × 18 columns (one row per customer, unique `Customer ID`) |
| Geography | 50 US states |
| Categories | Clothing, Accessories, Footwear, Outerwear |

**Columns**

| Group | Fields |
|---|---|
| Demographics | Customer ID, Age (18–70), Gender, Location, Subscription Status |
| Purchase details | Item Purchased, Category, Purchase Amount (USD, $20–$100), Size, Color, Season |
| Behaviour | Discount Applied, Promo Code Used, Previous Purchases (1–50), Frequency of Purchases, Review Rating (2.5–5.0), Shipping Type, Payment Method | 

**Data quality checks and cleaning**

| Issue | Action |
|---|---|
| 37 missing values in `Review Rating` (about 1%) | Imputed with the **median rating of each product category**, so one category's ratings do not skew another's |
| Inconsistent column names | Converted to `snake_case` (e.g. `Purchase Amount (USD)` → `purchase_amount`) |
| `Discount Applied` and `Promo Code Used` identical in all 3,900 rows | Confirmed redundant and **dropped `promo_code_used`** |
| No age grouping | Created `age_group` with quartile binning (`pd.qcut`): Young Adult 18–31, Adult 32–44, Middle-aged 45–57, Senior 58–70 |
| Purchase frequency stored as text | Created `purchase_frequency_days` (e.g. Weekly = 7, Monthly = 30, Annually = 365) |

---

## 🛠 Approach and Tools

1. **Python (pandas):** loading, profiling, cleaning and feature engineering (`Customer_Shopping_Behavior-Analysis.ipynb`).
2. **PostgreSQL:** the cleaned dataframe was loaded into a database through SQLAlchemy, then business questions were answered with SQL using aggregations, CTEs, subqueries, `CASE` and window functions (`Customer_Behavior_SQL_QUERIES.sql`).
3. **Power BI:** an interactive dashboard connected to the PostgreSQL table (`Customer_Behavior_Dashboard.pbix`).

---

## 📊 Analysis Results

**Overall:** total revenue of **$233,081**, average purchase of **$59.76**, average review rating of about **3.75 / 5**.

### 1. Revenue by gender

| Gender | Customers | Total revenue | Share of revenue | Avg purchase |
|---|---|---|---|---|
| Male | 2,652 | $157,890 | 67.7% | $59.54 |
| Female | 1,248 | $75,191 | 32.3% | $60.25 |


**Insight:** men generate about two-thirds of revenue, but only because they are about two-thirds of the customer base. **Average spend per purchase is nearly identical** ($59.54 vs. $60.25).

### 2. Revenue by category

| Category | Revenue | Share |
|---|---|---|
| Clothing | $104,264 | 44.7% |
| Accessories | $74,200 | 31.8% |
| Footwear | $36,093 | 15.5% |
| Outerwear | $18,524 | 7.9% |


**Insight:** Clothing and Accessories together bring in about 76% of revenue. Outerwear is the smallest category, with a lower average purchase ($57.17 vs. about $60 elsewhere).

### 3. Revenue by age group

| Age group | Revenue | Share | Avg purchase |
|---|---|---|---|
| Young Adult (18–31) | $62,143 | 26.7% | $60.45 |
| Middle-aged (45–57) | $59,197 | 25.4% | $60.04 |
| Adult (32–44) | $55,978 | 24.0% | $59.42 |
| Senior (58–70) | $55,763 | 23.9% | $59.07 |



**Insight:** revenue is spread fairly evenly across age groups. Young Adults lead, but the gap between the top and bottom group is under 3 percentage points.

### 4. Subscribers vs. non-subscribers

| Status | Customers | Avg spend | Total revenue |
|---|---|---|---|
| Non-subscribers | 2,847 (73%) | $59.87 | $170,436 |
| Subscribers | 1,053 (27%) | $59.49 | $62,645 |


**Insight:** subscribers do **not** spend more per purchase. Non-subscribers bring in more total revenue only because there are far more of them.

**Notable pattern:** in this dataset **none of the 1,248 female customers are subscribers**, while 39.7% of male customers are. Subscription is effectively a male-only feature in this data (see Limitations).

### 5. Customer loyalty segments

Segments are based on `previous_purchases`: **New** = 1, **Returning** = 2–10, **Loyal** = 11 or more.

| Segment | Customers | Share |
|---|---|---|
| Loyal | 3,116 | 79.9% |
| Returning | 701 | 18.0% |
| New | 83 | 2.1% |


**Insight:** the customer base is overwhelmingly repeat buyers. Only 2% are first-time purchasers, which points to strong retention but a weak flow of new customers.

### 6. Are repeat buyers more likely to subscribe?

Among customers with more than 5 previous purchases: **958 of 3,476 (27.6%)** subscribe, compared with **27.0%** across all customers.

**Insight:** repeat buying has essentially **no link to subscribing**. Frequent buyers are not being converted into subscribers.

### 7. Discounts

- **43%** of purchases (1,677) used a discount.
- Average purchase with a discount: **$59.28**, without: **$60.13**. Discounts did not lift spend.
- 839 of the 1,677 discounted purchases were at or above the overall average purchase amount (the "discount but still spent more than average" group).

**Products most dependent on discounts (share of purchases with a discount applied):**

| Rank | Product | Discount rate |
|---|---|---|
| 1 | Hat | 50.0% |
| 2 | Sneakers | 49.7% |
| 3 | Coat | 49.1% |
| 4 | Sweater | 48.2% |
| 5 | Pants | 47.4% |

### 8. Product performance

**Top 5 products by average review rating**

| Rank | Product | Avg rating |
|---|---|---|
| 1 | Gloves | 3.86 |
| 2 | Sandals | 3.84 |
| 3 | Boots | 3.82 |
| 4 | Hat | 3.80 |
| 5 | Handbag | 3.78 |

**Top 3 most purchased products per category**

| Category | #1 | #2 | #3 |
|---|---|---|---|
| Accessories | Jewelry (171) | Sunglasses (161) | Belt (161) |
| Clothing | Blouse (171) | Pants (171) | Shirt (169) |
| Footwear | Sandals (160) | Shoes (150) | Sneakers (145) |
| Outerwear | Jacket (163) | Coat (161) | n/a (only 2 products) |

**Insight:** ratings are tightly clustered (3.78 to 3.86 for the top 5), so product quality perception is similar across the range. Hats are the only product that appears in both the top-rated list and the most discount-reliant list.

### 9. Shipping type

| Shipping type | Avg purchase | Customers |
|---|---|---|
| 2-Day Shipping | $60.73 | 627 |
| Express | $60.48 | 646 |
| Free Shipping | $60.41 | 675 |
| Store Pickup | $59.89 | 650 |
| Next Day Air | $58.63 | 648 |
| Standard | $58.46 | 654 |

**Insight:** Express customers spend about **$2.02 (3.5%) more** than Standard customers. The difference is small compared with the typical spread in purchase amounts (standard deviation of about $24).

---

## 📈 Power BI Dashboard
The dashboard connects to the PostgreSQL `customer` table and includes:

- **KPI cards:** average purchase amount, average review rating, number of customers
- **Donut chart:** share of customers by subscription status
- **Column charts:** revenue by category and customer count by category
- **Bar charts:** revenue by age group and customer count by age group
- **Slicers:** subscription status, gender, category and shipping type, so every visual can be filtered interactively

---

## ✅ Recommendations

1. **Make subscriptions worth having, and open them to everyone.** Subscribers spend no more than non-subscribers, so the current offer is not changing behaviour. Add real member benefits (early access, free shipping thresholds, member pricing) and test whether they raise spend or purchase frequency. Investigate why no female customers are subscribed and treat that group as the largest untapped subscription opportunity.
2. **Convert loyal buyers into subscribers.** Four out of five customers are already loyal, yet they subscribe at the same rate as everyone else. Target them with a subscription offer at the point of purchase.
3. **Review the discount policy before spending more on it.** Discounted orders were not larger than full-price ones, and nearly half of Hat, Sneaker, Coat, Sweater and Pants purchases are discounted. Run a test where selected products are sold at full price to see whether volume holds, and protect margin.
4. **Invest in acquiring new customers.** Only 2% of customers are new. Retention looks healthy, but growth will stall without a pipeline of first-time buyers (referral offers, welcome incentives).
5. **Lead campaigns with proven products.** Feature top-rated items (Gloves, Sandals, Boots) and best sellers (Jewelry, Blouse, Pants, Sandals, Jacket) in marketing. Use ratings as social proof.
6. **Keep targeting broad rather than narrow.** Spend per purchase is similar across age groups, genders and shipping types, so the data does not support heavy targeting of one segment. Young Adults and Express/2-Day shipping users show slightly higher spend and are reasonable places to start a test, not to commit the budget.

---

## ⚠️ Limitations

- **No time dimension.** The data has no purchase dates, so trends, seasonality over time and growth cannot be analysed. Each customer appears once, which makes it a snapshot rather than a transaction history.
- **No cost or margin data.** Discount and shipping recommendations cannot be judged on profitability, only on revenue.
- **Very small differences between groups.** Most spend comparisons differ by only $1 to $3. No statistical significance tests were run, so these differences may be noise.
- **Subscription pattern looks unusual.** 0% of female customers subscribe versus 39.7% of men. This may be a data collection issue or a feature of the source dataset, and should be verified before acting on it.
- **Segment thresholds are skewed.** With `previous_purchases` ranging only from 1 to 50, the New/Returning/Loyal cut-offs put 80% of customers in "Loyal". Different thresholds (for example quartiles) would give a more useful split.
- **Imputation.** 37 missing review ratings were filled with category medians, which slightly smooths variation in ratings.
- **Quartile age groups** are based on this dataset's age distribution, not standard demographic bands.
- **Possible synthetic or sampled data.** Evenly spread values (purchase amounts, frequencies, ratings) suggest the data may not reflect a real customer base, so findings are illustrative.

---

## 📁 Repository Structure

```
├── Customer_Shopping_Behavior.csv              # Raw dataset
├── Customer_Shopping_Behavior-Analysis.ipynb   # Python cleaning and feature engineering
├── Customer_Behavior_SQL_QUERIES.sql           # PostgreSQL business queries
├── Customer_Behavior_Dashboard.pbix            # Power BI dashboard
├── Customer_Behavior_Report.docx               # Written project summary
├── images/                                     # Charts and dashboard screenshots
└── README.md
```

