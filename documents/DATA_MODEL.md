# 📐 Data Model Documentation

## Architecture Overview

This project uses the **Medallion Architecture** pattern, specifically the **Gold Layer** — the final, analytics-ready layer of the data pipeline. Data flows from raw sources through transformation layers before landing in the star schema below.

```
[Raw Sources / Bronze] → [Cleaned / Silver] → [Star Schema / Gold] → [Power BI]
```

---

## Entity Relationship Diagram (Conceptual)

```
                          ┌─────────────────────┐
                          │   gold_fact_orders   │ ← Central Fact Table
                          │---------------------|
                          │ order_line_id (PK)   │
                          │ order_date           │
                          │ revenue              │
              ┌───────────│ customer_id (FK)     │───────────┐
              │           │ product_id (FK)      │           │
              │     ┌─────│ seller_id (FK)       │─────┐     │
              │     │     │ payment_id (FK)      │     │     │
              │     │     │ campaign_id (FK)     │     │     │
              │     │     │ fulfillment_id (FK)  │     │     │
              │     │     │ delivery_person_id   │     │     │
              │     │     └──────────────────────┘     │     │
              │     │                                   │     │
              ▼     ▼                                   ▼     ▼
   ┌──────────────┐ ┌────────────────┐  ┌───────────────┐ ┌────────────────┐
   │dim_customer  │ │  dim_product   │  │  dim_seller   │ │  dim_payment   │
   └──────────────┘ └────────────────┘  └───────────────┘ └────────────────┘

         ┌────────────────────────┐   ┌──────────────────────────┐
         │  dim_delivery_person   │   │      dim_campaign         │
         └────────────────────────┘   └──────────────────────────┘

   ┌───────────────────┐   ┌──────────────────────────────┐
   │  gold_fact_returns │   │ gold_fact_fulfillment_pe     │
   └───────────────────┘   └──────────────────────────────┘
```

---

## Table Definitions

### 🔶 Fact Tables

---

#### `gold_fact_orders`
**Grain:** One row per order line item

| Column | Type | Description |
|--------|------|-------------|
| `order_line_id` | Text | Primary key — unique order line identifier |
| `order_date` | Date | Date the order was placed |
| `order_status` | Text | Current status (Completed, Cancelled, Returned, etc.) |
| `customer_id` | Text | FK → gold_dim_customer |
| `product_id` | Text | FK → gold_dim_product |
| `seller_id` | Text | FK → gold_dim_seller |
| `payment_id` | Text | FK → gold_dim_payment |
| `campaign_id` | Text | FK → gold_dim_campaign |
| `fulfillment_id` | Text | FK → gold_fact_fulfillment_pe |
| `delivery_person_id` | Text | FK → gold_dim_delivery_person |
| `quantity` | Integer | Number of units ordered |
| `unit_price` | Decimal | Price per unit |
| `gross_amount` | Decimal | Revenue before discounts |
| `discount_amount` | Decimal | Total discount applied |
| `discount_percentage` | Decimal | Discount as % of gross |
| `net_amount` | Decimal | Gross minus discount |
| `tax_amount` | Decimal | Tax charged |
| `tax_percentage` | Decimal | Tax rate applied |
| `shipping_fee` | Decimal | Shipping cost charged |
| `revenue` | Decimal | Final recognized revenue |
| `refund_amount` | Decimal | Amount refunded (if applicable) |
| `return_flag` | Boolean | 1 = returned, 0 = not returned |
| `return_rate` | Decimal | Return rate metric |
| `returns_rate` | Decimal | Aggregated returns rate |
| `avg_order_value` | Decimal | Average order value for the customer |
| `delivery_delay_days` | Integer | Days delayed beyond expected delivery |
| `delivery_sla_days` | Integer | SLA commitment in days |
| `expected_delivery_date` | Date | Promised delivery date |
| `actual_delivery_date` | Date | Actual delivery date |
| `delivery_rating` | Decimal | Customer rating of delivery experience |
| `customer_rating` | Decimal | Overall order satisfaction rating |
| `sentiment_score` | Decimal | NLP-derived sentiment from reviews |
| `prev_month` | Decimal | Revenue from previous month (for MoM) |

---

#### `gold_fact_returns`
**Grain:** One row per returned order line

| Column | Type | Description |
|--------|------|-------------|
| `order_line_id` | Text | FK → gold_fact_orders |
| `customer_id` | Text | FK → gold_dim_customer |
| `product_id` | Text | FK → gold_dim_product |
| `payment_id` | Text | FK → gold_dim_payment |
| `fulfillment_id` | Text | FK → gold_fact_fulfillment_pe |
| `return_date` | Date | Date the return was initiated |
| `days_to_return` | Integer | Days between purchase and return |
| `refund_amount` | Decimal | Amount refunded to customer |
| `restocking_fee` | Decimal | Fee charged for restocking |
| `unit_price` | Decimal | Original unit price |
| `quantity` | Integer | Units returned |
| `customer_rating` | Decimal | Rating given at time of return |
| `sentiment_score` | Decimal | Sentiment of return review |

---

#### `gold_fact_fulfillment_pe`
**Grain:** One row per fulfillment record (delivery agent + shipment)

| Column | Type | Description |
|--------|------|-------------|
| `fulfillment_id` | Text | Primary key |
| `delivery_person_id` | Text | FK → gold_dim_delivery_person |
| `service_level` | Text | Express / Standard / Economy |
| `shipping_method` | Text | Courier, Postal, etc. |
| `delivery_sla_days` | Integer | SLA commitment |
| `avg_delivery_delay_days` | Decimal | Average days delayed |
| `avg_delivery_rating` | Decimal | Average delivery satisfaction |
| `base_shipping_cost` | Decimal | Base cost before adjustments |
| `total_shipping_cost` | Decimal | Final shipping cost |
| `total_deliveries` | Integer | Total deliveries handled |
| `completed_deliveries` | Integer | Successfully delivered |
| `on_time_deliveries` | Integer | Delivered within SLA |
| `late_deliveries` | Integer | Delivered past SLA |
| `returned_deliveries` | Integer | Returned after delivery |
| `return_rate` | Decimal | Return rate for this fulfillment node |
| `sla_breach_rate` | Decimal | % of deliveries that breached SLA |

---

### 🔷 Dimension Tables

---

#### `gold_dim_customer`

| Column | Description |
|--------|-------------|
| `customer_id` | Primary key |
| `customers_name` | Full name |
| `gender` | M / F |
| `age` | Age in years |
| `date_of_birth` | DOB |
| `email` | Contact email |
| `phone_number` | Contact number |
| `city`, `state`, `region` | Geographic location |
| `postal_code`, `location_id` | Address details |
| `marital_status` | Single / Married / etc. |
| `income_bracket` | Low / Lower-Middle / Middle / Upper-Middle / High |
| `first_purchase_date` | Date of first order |
| `last_purchase_date` | Date of most recent order |
| `recency_days` | Days since last purchase |
| `frequency_orders` | Total number of orders placed |
| `monetary_value` | Total spend (RFM model value) |
| `avg_order_value` | Average spend per order |

---

#### `gold_dim_product`

| Column | Description |
|--------|-------------|
| `product_id` | Primary key |
| `product_name` | Product display name |
| `category` | Electronics, Books, Groceries, Clothing, Home |
| `sub_category` | Sub-level classification |
| `brand` | Brand name |
| `list_price` | Standard listed price |

---

#### `gold_dim_seller`

| Column | Description |
|--------|-------------|
| `seller_id` | Primary key |
| `seller_name` | Business name |
| `city`, `state`, `region` | Seller location |
| `postal_code`, `location_id` | Address details |
| `email`, `phone_number` | Contact info |
| `category_focus` | Primary category the seller operates in |
| `rating` | Seller rating score |
| `join_date` | Date seller joined the platform |

---

#### `gold_dim_payment`

| Column | Description |
|--------|-------------|
| `payment_id` | Primary key |
| `payment_method` | Credit Card, Wallet, COD, etc. |
| `payment_provider` | Visa, Mastercard, PayPal, etc. |
| `description` | Additional details |

---

#### `gold_dim_delivery_person`

| Column | Description |
|--------|-------------|
| `delivery_person_id` | Primary key |
| `delivery_person_name` | Full name |
| `gender` | M / F |
| `city`, `state`, `region` | Operating area |
| `postal_code`, `location_id` | Location details |
| `phone_number` | Contact number |
| `employment_type` | Full-time / Freelance / Contract |
| `vehicle_type` | Bike, Car, Van, etc. |
| `date_of_joining` | Joining date |

---

#### `gold_dim_campaign`

| Column | Description |
|--------|-------------|
| `campaign_id` | Primary key |
| `campaign_name` | Name of marketing campaign |
| `campaign_type` | Type (Retargeting, Awareness, etc.) |
| `channel_name` | Instagram, Google, Email, etc. |
| `channel_type` | Paid / Organic / Social |
| `objective` | Campaign goal |
| `budget` | Total allocated budget |
| `spend_amount` | Amount actually spent |
| `clicks` | Total clicks generated |
| `impressions` | Total impressions |
| `conversions` | Conversions attributed |
| `revenue_generated` | Revenue from campaign |
| `avg_order_values` | Avg order value from campaign traffic |
| `revenue_cur_month` | Campaign revenue this month |
| `revenue_prev_month` | Campaign revenue previous month |
| `revenue_MOM` | Month-over-Month growth rate |

---

## Relationships Summary

| From Table | From Column | To Table | To Column | Cardinality |
|-----------|-------------|----------|-----------|-------------|
| gold_fact_orders | customer_id | gold_dim_customer | customer_id | Many-to-One |
| gold_fact_orders | product_id | gold_dim_product | product_id | Many-to-One |
| gold_fact_orders | seller_id | gold_dim_seller | seller_id | Many-to-One |
| gold_fact_orders | payment_id | gold_dim_payment | payment_id | Many-to-One |
| gold_fact_orders | campaign_id | gold_dim_campaign | campaign_id | Many-to-One |
| gold_fact_orders | delivery_person_id | gold_dim_delivery_person | delivery_person_id | Many-to-One |
| gold_fact_orders | fulfillment_id | gold_fact_fulfillment_pe | fulfillment_id | Many-to-One |
| gold_fact_returns | customer_id | gold_dim_customer | customer_id | Many-to-One |
| gold_fact_returns | product_id | gold_dim_product | product_id | Many-to-One |
| gold_fact_returns | payment_id | gold_dim_payment | payment_id | Many-to-One |
| gold_fact_returns | order_line_id | gold_fact_orders | order_line_id | Many-to-One |
| gold_fact_fulfillment_pe | delivery_person_id | gold_dim_delivery_person | delivery_person_id | Many-to-One |

---

## Design Decisions

1. **Star Schema over Snowflake** — Denormalized for Power BI performance; avoids multi-hop joins which slow DAX queries.
2. **Separate Returns Fact Table** — Returns have different grain and attributes from orders; combining them would create sparse columns and complicate measures.
3. **Fulfillment as a Fact Table** — Fulfillment has its own aggregatable metrics (SLA breach rate, delivery ratings) making it better suited as a fact than a dimension.
4. **RFM fields in dim_customer** — Recency, Frequency, and Monetary values stored pre-computed for performance.
5. **Campaign metrics in dim_campaign** — Campaign performance data (MoM, revenue) pre-aggregated to avoid expensive calculations at query time.
