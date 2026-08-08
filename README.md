# Lenskart Growth and Sales Analytics

## 📌 Overview
A data analytics case study analyzing Lenskart's customer and sales data to identify growth trends and opportunities. The project covers data cleaning, exploratory analysis, and reporting to support business decisions on customer acquisition and retention.

## 🎯 Business Problem
Lenskart wanted to understand what's driving (or holding back) customer acquisition and retention, and to surface growth opportunities in their sales data that could support business decision-making.

## 🗂️ Dataset
- **Source:** [https://www.kaggle.com/datasets/datamonkresearch/lenskart-india-sales-and-customer-analytics-dataset]
- **Tables/Files:** `orders_normalized` (fact table), `customers`, `products`, `locations`
-  spanning 2022–2025

## 🛠️ Tools & Tech Stack
- **SQL** (MySQL Workbench) — data extraction and business-question queries
- **Python** (Pandas, Matplotlib/Seaborn, SciPy, scikit-learn, statsmodels) — EDA, statistical testing, clustering, regression, and forecasting
- **Power BI** — interactive sales performance dashboard

## 🔑 Key Insights
- **Discounting isn't growing basket size, it's eroding margin** — quantity per order stays flat (~2 items) at every discount level, while average order value drops from ₹5,827 (0% discount) to ₹4,385 (25% discount).
- **Growth is real but decelerating** — YoY growth slowed from 3.89% (2024) to 1.43% (2025), with a linear forecast projecting 2026 revenue to nearly flatten.
- **Today's biggest markets aren't tomorrow's fastest-growing ones** — Maharashtra leads in current revenue, but Bihar (+46.3%) and Chandigarh (+38.5%) are growing far faster and warrant more investment.

## 📁 Repository Structure
```
├── README.md
├── sql/
│   └── sql.md                            # Queries + explanations + answers
├── notebooks/
│   └── Lenskart_Data_Analysis.ipynb       # Python EDA, plots, findings
├── powerbi/                              # if applicable
│   ├── Lenskart_Data_Dashboard.pbix
│   
```

## 📊 Dashboard Preview
<img width="1181" height="663" alt="image" src="https://github.com/user-attachments/assets/62d36f58-053d-4e1a-b433-12d00d75c66a" />



## 👤 Author
Arjav Jain
