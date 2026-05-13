# 🛒 E-Commerce Analytics Dashboard

> **A full end-to-end data analytics project built with Power BI** — covering data modeling, DAX measures, and interactive business intelligence reporting for an e-commerce business.

<img width="1510" height="851" alt="image" src="https://github.com/user-attachments/assets/5c7671e9-393b-4290-a4f3-f421ee160f3c" />


---

## 📊 Project Overview

This project simulates a real-world e-commerce analytics use case. Starting from raw transactional data, I designed a **star-schema data warehouse** (Gold layer), wrote **DAX measures** for business KPIs, and built an interactive **Power BI dashboard** that stakeholders can use to monitor performance across sales, returns, fulfillment, and customer segments.

### Key Business Questions Answered
- 💰 How much revenue are we generating, and how is it trending day over day?
- 📦 Which product categories drive the most orders — and the most returns?
- 🏆 Who are our top-performing sellers?
- 🌍 Which regions contribute the most revenue?
- 👥 How are orders distributed across customer income segments?

---

## 🎯 Dashboard Highlights

| KPI | Value |
|-----|-------|
| **Total Revenue** | 34.18 Billion |
| **Total Orders** | 790.73K |
| **Total Returns** | 103.04K |
| **Avg. Order Value** | 43.22K |
| **Returns Rate** | 13.03% |

### Visuals Included
- **Orders by Category** — Bar chart (Electronics leads at 63K)
- **Top Sellers** — Ranked horizontal bar chart (Wali Group #1 with 794 orders)
- **Revenue by Region** — Horizontal bar chart (North leads at 9.6bn)
- **Revenue per Day** — Combo chart (bars + line trend) showing daily order count and revenue
- **Return by Category** — Horizontal bar chart (Electronics highest at 8.2K)
- **Order by Customer Segmentation** — Treemap across Middle, Lower-Middle, Low, Upper-Middle income brackets
- **Month slicer** — Dynamic filtering across all visuals

---

## 🗄️ Data Model

The data model follows a **Medallion Architecture (Gold Layer)** — a star schema optimized for analytical queries.

```
gold_fact_orders          ← Central fact table
gold_fact_returns         ← Returns fact table
gold_fact_fulfillment_pe  ← Fulfillment performance fact table

gold_dim_product          ← Product dimension
gold_dim_customer         ← Customer dimension
gold_dim_seller           ← Seller dimension
gold_dim_payment          ← Payment method dimension
gold_dim_delivery_person  ← Delivery agent dimension
gold_dim_campaign         ← Marketing campaign dimension
```

### Fact Tables

#### `gold_fact_orders`
The central fact table storing one row per order line. Key fields:
- `order_date`, `order_status`, `order_line_id`
- `revenue`, `gross_amount`, `net_amount`, `unit_price`, `quantity`
- `discount_amount`, `discount_percentage`, `tax_amount`, `tax_percentage`
- `shipping_fee`, `refund_amount`, `return_flag`, `return_rate`, `returns_rate`
- `delivery_delay_days`, `delivery_sla_days`, `delivery_rating`, `customer_rating`
- `sentiment_score`, `avg_order_value`, `prev_month`
- FK: `customer_id`, `seller_id`, `product_id`, `payment_id`, `campaign_id`, `fulfillment_id`, `delivery_person_id`

#### `gold_fact_returns`
Tracks returned orders:
- `days_to_return`, `refund_amount`, `restocking_fee`, `unit_price`, `quantity`
- `sentiment_score`, `customer_rating`
- FK: `customer_id`, `product_id`, `payment_id`, `fulfillment_id`, `order_line_id`

#### `gold_fact_fulfillment_pe`
Tracks delivery and fulfillment performance:
- `avg_delivery_delay_days`, `avg_delivery_rating`, `base_shipping_cost`, `total_shipping_cost`
- `completed_deliveries`, `on_time_deliveries`, `late_deliveries`, `returned_deliveries`, `total_deliveries`
- `return_rate`, `sla_breach_rate`, `delivery_sla_days`, `service_level`, `shipping_method`
- FK: `delivery_person_id`, `fulfillment_id`

### Dimension Tables

| Table | Key Fields |
|-------|-----------|
| `gold_dim_product` | `product_id`, `product_name`, `category`, `sub_category`, `brand`, `list_price` |
| `gold_dim_customer` | `customer_id`, `customers_name`, `gender`, `age`, `city`, `region`, `state`, `income_bracket`, `marital_status`, `monetary_value`, `recency_days`, `frequency_orders`, `avg_order_value` |
| `gold_dim_seller` | `seller_id`, `seller_name`, `city`, `region`, `state`, `category_focus`, `rating` |
| `gold_dim_payment` | `payment_id`, `payment_method`, `payment_provider`, `description` |
| `gold_dim_delivery_person` | `delivery_person_id`, `delivery_person_name`, `city`, `region`, `state`, `gender`, `employment_type`, `vehicle_type` |
| `gold_dim_campaign` | `campaign_id`, `campaign_name`, `campaign_type`, `channel_name`, `channel_type`, `objective`, `budget`, `spend_amount`, `clicks`, `impressions`, `conversions`, `revenue_generated`, `avg_order_values`, `revenue_cur_month`, `revenue_prev_month`, `revenue_MOM` |

---

## 🧮 Key DAX Measures

```dax
-- Total Revenue
Total Revenue = SUM(gold_fact_orders[revenue])

-- Total Orders
Total Orders = DISTINCTCOUNT(gold_fact_orders[order_line_id])

-- Total Returns
Total Returns = COUNTROWS(gold_fact_returns)

-- Average Order Value
Avg Order Value = AVERAGE(gold_fact_orders[avg_order_value])

-- Returns Rate
Returns Rate = DIVIDE([Total Returns], [Total Orders], 0)

-- Revenue per Day (used in combo chart)
Revenue per Day = CALCULATE(SUM(gold_fact_orders[revenue]), ALLEXCEPT(gold_fact_orders, gold_fact_orders[order_date]))
```

---

## 📁 Repository Structure

```
ecommerce-analytics-powerbi/
│
├── 📊 ECommerce_Dashboard.pbix       # Main Power BI file
│
├── 📂 data/
│   └── raw/                          # Raw CSV source files
│       ├── orders.csv
│       ├── returns.csv
│       ├── customers.csv
│       ├── products.csv
│       ├── sellers.csv
│       ├── payments.csv
│       ├── delivery_persons.csv
│       └── campaigns.csv
│
├── 📂 assets/
│   └── dashboard-preview.png         # Dashboard screenshot
│
├── 📂 docs/
│   └── DATA_MODEL.md                 # Detailed data model documentation
│
└── README.md
```

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| **Power BI Desktop** | Dashboard design, DAX measures, data modeling |
| **Power Query (M)** | Data transformation and cleaning inside Power BI |
| **CSV / Raw Data** | Source data loaded directly into Power BI |
| **Star Schema / Medallion Architecture** | Data warehouse design pattern |

---

## 🚀 How to Use

1. **Clone this repository**
   ```bash
   https://github.com/HendroCahyoRamadhan/E-CommerceDashboardBreakdownPowerBI
   ```

2. **Open the Power BI file**
   - Open `ECommerce_Dashboard.pbix` in **Power BI Desktop**

3. **Connect to data** *(if needed)*
   - If data source paths break, go to **Transform Data → Data Source Settings**
   - Re-point each CSV to its file under your local `/data/raw/` folder

4. **Explore the dashboard**
   - Use the **Month slicer** (top right) to filter all visuals by month
   - Hover over charts for detailed tooltips
   - Click on bars/segments for cross-filtering between visuals

---

## 📈 Insights & Findings

- **Electronics** dominates both orders (63K) and returns (8.2K) — high volume, high churn; worth investigating product quality or customer expectations.
- **North region** leads revenue at 9.6bn, almost double the **Central region** (2.1bn) — possible opportunity for regional campaigns.
- **Wali Group** is the top seller with 794 orders, outperforming competitors by ~7%.
- **Middle income** customers account for the largest order segment (0.28M), making them the primary target demographic.
- **Returns rate of 13%** is significant — a focused returns-reduction initiative could meaningfully improve margins.

---

## 👤 Author

**Hendro Cahyo Ramadhan**
- 📧 hendrocr20@gmail.com
- 🐙 (https://github.com/HendroCahyoRamadhan)

---

## 📜 License

This project is for portfolio and educational purposes. Data used is synthetic/anonymized.
