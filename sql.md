# 🩺 Lenskart Growth & Sales Analytics — SQL Case Study

The dataset was imported from CSV into MySQL Workbench, where the 4 tables below were formed for analysis.

## 🏗️ Table Structure

| Table Name | Description | Key Columns |
|---|---|---|
| `orders_normalized` | Fact table — one row per order, with sales, discount, and payment details | `order_id` (PK), `order_date`, `location_id` (FK), `product_id` (FK), `customer_id` (FK), `sales_channel`, `store_type`, `quantity`, `unit_price`, `discount_percent`, `net_sales_amount`, `payment_mode` |
| `customers` | Customer demographic details | `customer_id` (PK), `customer_gender`, `customer_age` |
| `products` | Product category lookup | `product_id` (PK), `product_category` |
| `locations` | Store location details | `location_id` (PK), `city`, `state` |
 
**Schema:** `orders_normalized` is the central fact table, linked to `customers`, `products`, and `locations` via foreign keys (`customer_id`, `product_id`, `location_id`).

**Screenshot of table structure :**

<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/90d5e4f7-355e-4dc6-8f84-0848f9380338" />


## ⚠️ A Note on `customer_id`
 
The `customers` table wasn't built from real customer identities — it was formed by pulling the **distinct combinations** of `customer_gender` and `customer_age` already present in the orders data:
 
```sql
INSERT INTO customers (customer_gender, customer_age)
SELECT DISTINCT customer_gender, customer_age FROM orders;
```
 
So `customer_id` here is really a stand-in for a **demographic combination** (e.g. "Female, age 36"), not a unique person. Every order from *any* 36-year-old female customer maps to the *same* `customer_id`, even if they were different people who never interacted.
 
**Why this matters:**
- ✅ **Fine to use for:** demographic questions — e.g. Q5 (customer segment value) and Q6 in the Python notebook (clustering) are still valid, since that's genuinely what's being measured.
- ❌ **Not valid for:** anything implying individual customer behavior — repeat purchase rate, customer lifetime value, or cohort/retention analysis. The dataset doesn't track a real customer across multiple orders, so those questions can't be answered from this data.

  

## 🔎 Analysis Queries



---

### Q1. Which regions and categories generate the most revenue?

```sql
SELECT l.state, p.product_category,
COUNT(*) AS total_orders,
ROUND(SUM(o.net_sales_amount), 2) AS total_revenue
FROM orders_normalized o
JOIN locations l ON o.location_id = l.location_id
JOIN products p ON o.product_id = p.product_id
GROUP BY l.state, p.product_category
ORDER BY total_revenue DESC limit 4;
```

**Result:**

<img width="452" height="123" alt="image" src="https://github.com/user-attachments/assets/bbc24cdd-5117-420c-b805-bffcb6e27b01" />


**Explanation:**

Kerala's Contact Lenses segment generates the highest revenue overall, closely followed by Maharashtra's Sunglasses. Contact Lenses appear twice in the top 4 (Kerala and Odisha), showing consistently strong demand for that category across states, while Maharashtra stands out as strong in two categories at once.

---

### Q2. TOP-SELLING PRODUCT CATEGORY PER STATE

```sql
SELECT ranked.state,ranked.product_category,ranked.revenue FROM (
SELECT l.state, p.product_category,
SUM(o.net_sales_amount) AS revenue,
RANK() OVER (PARTITION BY l.state ORDER BY SUM(o.net_sales_amount) DESC) AS rnk
FROM orders_normalized o
JOIN locations l ON o.location_id = l.location_id
JOIN products p ON o.product_id = p.product_id
GROUP BY l.state, p.product_category
) ranked
WHERE rnk = 1
ORDER BY revenue DESC;
```

**Result:**

<img width="335" height="136" alt="image" src="https://github.com/user-attachments/assets/6b6a6e8e-42ab-4875-a3a2-f03bd865f90c" />



**Explanation:**

The top category isn't the same everywhere — Contact Lenses lead in Kerala and Odisha, Sunglasses lead in Maharashtra and Tamil Nadu, and Madhya Pradesh's top category is Computer Glasses. This shows category preference is state-specific, so a single national stocking strategy would underperform a state-tailored one.

---

### Q3. Where should the business invest — online, or which offline format?

```sql
SELECT sales_channel, store_type,
COUNT(*) AS total_orders,
ROUND(AVG(net_sales_amount), 2) AS avg_order_value,
ROUND(SUM(net_sales_amount), 2) AS total_revenue
FROM orders_normalized
GROUP BY sales_channel, store_type
ORDER BY total_revenue DESC;
```

**Result:**

<img width="547" height="137" alt="image" src="https://github.com/user-attachments/assets/a06e1023-8c4e-4a4e-9258-ceb4517ebbd5" />


**Explanation:**

Online drives nearly 4x the revenue of any single offline format, purely on order volume (2026 orders vs ~500 each offline). But per-order value tells a different story — Standalone stores have the highest average order value (₹5517) of all formats, including online, meaning offline still converts higher-value baskets even at lower volume.

---
### Q4. DOES DISCOUNT LEVEL ACTUALLY DRIVE BIGGER BASKETS?

```sql
SELECT discount_percent,
COUNT(*) AS total_orders,
ROUND(AVG(quantity), 2) AS avg_quantity,
ROUND(AVG(net_sales_amount), 2) AS avg_net_sales
FROM orders_normalized
GROUP BY discount_percent
ORDER BY discount_percent;
```

**Result:**

<img width="427" height="156" alt="image" src="https://github.com/user-attachments/assets/9356609e-3650-496d-83c4-810b0c0ffddb" />



**Explanation:**

Average quantity per order stays flat at ~2 items regardless of discount level, but average net sales steadily drops as discounts increase (₹5827 at 0% down to ₹4385 at 25%). This means higher discounts aren't encouraging customers to buy more — they're just cutting into revenue per order, so the discounting strategy is eroding margin rather than driving bigger baskets.

---
### Q5. Which demographic should marketing prioritize?

```sql
SELECT
CASE
WHEN c.customer_age < 25 THEN '<25'
WHEN c.customer_age BETWEEN 25 AND 34 THEN '25-34'
WHEN c.customer_age BETWEEN 35 AND 44 THEN '35-44'
WHEN c.customer_age BETWEEN 45 AND 54 THEN '45-54'
ELSE '55+'
END AS age_band,
c.customer_gender,
COUNT(*) AS total_orders,
ROUND(SUM(o.net_sales_amount), 2) AS total_revenue
FROM orders_normalized o
JOIN customers c ON o.customer_id = c.customer_id
GROUP BY age_band, c.customer_gender
ORDER BY total_revenue DESC ,total_orders DESC limit 5;
```

**Result:**

<img width="405" height="137" alt="image" src="https://github.com/user-attachments/assets/6c136537-3209-40bd-b355-bb45a2a2c856" />



**Explanation:**

Older age bands (45+) dominate the top revenue segments, with 55+ Female customers generating the most revenue overall. Younger customers (under 25 and 25-34) don't appear in the top segments except one Other-gender group, suggesting marketing spend aimed at older demographics is likely to yield better returns than targeting younger buyers.

---
### Q6. STATES BEATING THE OVERALL AVERAGE ORDER VALUE(Which regions are punching above their weight?)

```sql
SELECT l.state,
ROUND(AVG(o.net_sales_amount), 2) AS avg_order_value
FROM orders_normalized o
JOIN locations l ON o.location_id = l.location_id
GROUP BY l.state
HAVING AVG(o.net_sales_amount) > (
SELECT AVG(net_sales_amount) FROM orders_normalized
)
ORDER BY avg_order_value DESC;
```

**Result:**

<img width="240" height="235" alt="image" src="https://github.com/user-attachments/assets/5aaa1dd8-633c-4ed5-be13-dc2e690ab321" />



**Explanation:**

The overall average order value across all orders is ₹5257.53. Rajasthan tops the list at ₹5804.98, well above average, followed by West Bengal and Chandigarh. Notably, smaller or non-metro states like Rajasthan and Chandigarh outperform larger markets like Maharashtra, suggesting these regions carry higher-value customers per order despite lower overall order volume.

---
### Q7. YEAR-OVER-YEAR REVENUE TREND (Is the business growing, and how fast?)

```sql
WITH yearly AS (
SELECT DATE_FORMAT(order_date, '%Y') AS year,
SUM(net_sales_amount) AS revenue
FROM orders_normalized
GROUP BY year
)
SELECT year, revenue,
LAG(revenue) OVER (ORDER BY year) AS prev_year_revenue,
ROUND(
(revenue - LAG(revenue) OVER (ORDER BY year)) * 100.0
/ LAG(revenue) OVER (ORDER BY year), 2
) AS year_growth_pct
FROM yearly
ORDER BY year;
```

**Result:**

<img width="407" height="122" alt="image" src="https://github.com/user-attachments/assets/179d374a-a5f4-4d91-bf4e-897fd1d52d07" />



**Explanation:**

Revenue dipped 3.60% in 2023 before recovering with 3.89% growth in 2024, then growth slowed to 1.43% in 2025. So while the business bounced back after a rough 2023, growth momentum is decelerating year over year rather than accelerating — a signal that current growth levers may be losing effectiveness.

---
### Q8. TOP CITY BY REVENUE WITHIN EACH STATE(Where should a flagship store go, state by state?)

```sql
SELECT * FROM (
SELECT l.state, l.city,
SUM(o.net_sales_amount) AS revenue,
RANK() OVER (PARTITION BY l.state ORDER BY SUM(o.net_sales_amount) DESC) AS rnk
FROM orders_normalized o
JOIN locations l ON o.location_id = l.location_id
GROUP BY l.state, l.city
) ranked
WHERE rnk = 1
ORDER BY revenue DESC limit 5;
```

**Result:**

<img width="328" height="141" alt="image" src="https://github.com/user-attachments/assets/8b671334-e326-4496-b327-d56c0fcb281f" />




**Explanation:**

Pune leads all cities in revenue, narrowly ahead of Hyderabad, with all top 5 cities crossing ₹10 lakh in revenue. These are the natural first candidates for a flagship store in their respective states — each is already the clear revenue leader within its state, showing concentrated demand rather than revenue spread thin across multiple cities.

---

## ✅ Conclusion
 
- **Discounting is eroding margin, not growing baskets** — average quantity per order stays flat (~2 items) at every discount level, while average revenue per order steadily drops as discount % increases (Q4). This is the clearest actionable finding in the dataset: the current discount strategy is a cost, not a growth lever.
- **Demand is regional, not universal** — the top-selling category, top city, and above-average states all differ by region (Q1, Q2, Q8). Stocking, marketing, and expansion decisions should be state/city-specific rather than following one national strategy.
- **Growth is real but decelerating** — revenue recovered after a 2023 dip, but YoY growth slowed from 3.89% (2024) to 1.43% (2025) (Q7), signalling current growth drivers are losing steam.
- **The 45+ age band is the highest-value customer segment** (Q5), outperforming younger buyers across revenue — suggesting marketing spend is better directed there than at a broader or younger audience.
  
**Overall:** the analysis points to a business with real but slowing growth, where the biggest near-term lever isn't acquiring more customers — it's fixing a discount strategy that's quietly eating margin, and focusing stocking/expansion on the states, cities, and age segments already proven to perform best.

