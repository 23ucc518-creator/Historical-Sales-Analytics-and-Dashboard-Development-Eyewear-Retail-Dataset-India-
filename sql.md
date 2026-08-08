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

**Screenshot of table structure / sample rows:**

<img width="100" height="500" alt="image" src="https://github.com/user-attachments/assets/90d5e4f7-355e-4dc6-8f84-0848f9380338" />


## 🔎 Analysis Queries

*(Add your queries and explanations below, following the format: Question → Query → Result → Explanation)*

---

### Q1. [Business question]

```sql
-- your query here
```

**Result:**

[📸 attach result screenshot / table]

**Explanation:**

[your explanation here]

---
