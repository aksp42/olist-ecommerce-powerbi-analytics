# 🧩 Olist Brazilian E-Commerce Public Dataset

> Source dataset for the [Olist E-Commerce Analytics — Power BI Portfolio Project](../README.md).
> This file documents the raw data used to build the data model, so the repository does not need to ship the large CSV files themselves.

![Dataset](https://img.shields.io/badge/Dataset-Olist-orange)
![Records](https://img.shields.io/badge/Orders-~100K-blue)
![Period](https://img.shields.io/badge/Period-2016--2018-green)
![Source](https://img.shields.io/badge/Source-Kaggle-20BEFF?logo=kaggle&logoColor=white)

---

## 📑 Table of Contents

- [Overview](#-overview)
- [How to Get the Data](#-how-to-get-the-data)
- [Data Schema](#-data-schema)
- [Table Reference](#-table-reference)
- [Primary Relationships](#-primary-relationships)
- [Data Characteristics](#-data-characteristics)
- [Key Analysis Areas](#-key-analysis-areas)
- [Business Use Cases](#-business-use-cases)
- [Known Limitations](#-known-limitations)
- [Acknowledgement & License](#-acknowledgement--license)

---

## 🚀 Overview

The **Olist Brazilian E-Commerce Public Dataset** contains approximately **100,000 anonymized orders** placed across multiple marketplaces in Brazil between **2016 and 2018**. It captures the full commercial lifecycle of an order — from purchase, payment and item-level detail through to carrier delivery and the customer's post-purchase review — along with customer, seller, product and geolocation attributes.

This dataset is the single source of truth behind every dashboard, DAX measure and calculated column documented in [`/Documentation`](../Documentation).

---

## 📥 How to Get the Data

The raw CSV files are **not committed to this repository** (per standard practice for large public datasets). To reproduce the model:

1. Download the dataset from Kaggle: **[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)**.
2. Extract all CSV files into this `Data/` folder (or point Power Query to wherever you've stored them).
3. Open `PowerBI/Olist_Ecommerce_Analytics.pbix` and refresh — the model's Power Query source paths can be repointed via **Transform Data → Data Source Settings**.

| File on Kaggle | Loaded as |
|---|---|
| `olist_orders_dataset.csv` | `olist_orders_dataset` |
| `olist_order_items_dataset.csv` | `olist_order_items_dataset` |
| `olist_order_payments_dataset.csv` | `olist_order_payments_dataset` |
| `olist_order_reviews_dataset.csv` | `olist_order_reviews_dataset` |
| `olist_customers_dataset.csv` | `olist_customers_dataset` |
| `olist_products_dataset.csv` | `olist_products_dataset` |
| `olist_sellers_dataset.csv` | `olist_sellers_dataset` |
| `olist_geolocation_dataset.csv` | `olist_geolocation_dataset` |
| `product_category_name_translation.csv` | `product_category_name_translation` |

---

## 🗺️ Data Schema

The dataset is split into multiple related tables, joined through `order_id`, `customer_id`, `product_id`, `seller_id`, and `zip_code_prefix`:

![Olist Data Schema](./Data_Schema.jpg)

```text
                 olist_order_payments_dataset          olist_products_dataset
                            ▲                                    ▲
                            │ order_id                product_id │
                            │                                    │
olist_order_reviews_dataset │                                    │
        ▲                  │                                    │
        │ order_id         │                                    │
        │                  │                                    │
        └──────────── olist_orders_dataset ──────────────► olist_order_items_dataset ◄──── olist_sellers_dataset
                            ▲                                                                        ▲
                            │ customer_id                                                zip_code_prefix
                            │                                                                        │
                    olist_customers_dataset ─────────── zip_code_prefix ───────────► olist_geolocation_dataset
```

> The diagram above mirrors the official Kaggle schema. In this project's Power BI model, `olist_order_customer_dataset` is loaded as **`olist_customers_dataset`** — see the [Table Reference](#-table-reference) below for the exact names used throughout the model and DAX library.

---

## 📚 Table Reference

| Table | Role | Grain | Key Fields |
|---|---|---|---|
| `olist_orders_dataset` | Order lifecycle (central fact table) | 1 row / order | `order_id`, `customer_id`, `order_status`, `order_purchase_timestamp`, `order_approved_at`, `order_delivered_carrier_date`, `order_delivered_customer_date`, `order_estimated_delivery_date` |
| `olist_order_items_dataset` | Line items per order | 1 row / product line | `order_id`, `order_item_id`, `product_id`, `seller_id`, `shipping_limit_date`, `price`, `freight_value` |
| `olist_order_payments_dataset` | Payment transactions | 1 row / payment sequence | `order_id`, `payment_sequential`, `payment_type`, `payment_installments`, `payment_value` |
| `olist_order_reviews_dataset` | Post-purchase reviews | 1 row / review | `review_id`, `order_id`, `review_score`, `review_comment_title`, `review_comment_message`, `review_creation_date`, `review_answer_timestamp` |
| `olist_customers_dataset` | Customer master | 1 row / `customer_id` | `customer_id`, `customer_unique_id`, `customer_zip_code_prefix`, `customer_city`, `customer_state` |
| `olist_products_dataset` | Product master | 1 row / product | `product_id`, `product_category_name`, `product_name_lenght`, `product_description_lenght`, `product_photos_qty`, `product_weight_g`, `product_length_cm`, `product_height_cm`, `product_width_cm` |
| `olist_sellers_dataset` | Seller master | 1 row / seller | `seller_id`, `seller_zip_code_prefix`, `seller_city`, `seller_state` |
| `olist_geolocation_dataset` | Zip-code-level geographic lookup | Many rows / zip prefix | `geolocation_zip_code_prefix`, `geolocation_lat`, `geolocation_lng`, `geolocation_city`, `geolocation_state` |
| `product_category_name_translation` | Category name lookup (PT → EN) | 1 row / category | `product_category_name`, `product_category_name_english` |

> 💡 `customer_id` is unique **per order**, while `customer_unique_id` identifies the same real-world shopper across multiple orders — always use `customer_unique_id` for true customer-level counts (repeat purchase rate, retention, etc.).

---

## 🔗 Primary Relationships

```text
Orders (order_id)      → Order Items (order_id)
Orders (order_id)      → Payments (order_id)
Orders (order_id)      → Reviews (order_id)
Orders (customer_id)   → Customers (customer_id)
Order Items (product_id) → Products (product_id)
Order Items (seller_id)  → Sellers (seller_id)
Customers (zip prefix)  → Geolocation (zip prefix)
Sellers (zip prefix)    → Geolocation (zip prefix)
Products (category)     → Category Name Translation (category)
```

A dedicated **Calendar** table (built in DAX from `Order Date`) drives all Year / Quarter / Half-Year / Month time intelligence — see [`Documentation/Olist_All_DAX_Measures_Columns_Tables.txt`](../Documentation/Olist_All_DAX_Measures_Columns_Tables.txt).

---

## 🔎 Data Characteristics

- One order may contain **multiple products** and be fulfilled by **multiple sellers**.
- Review scores range from **1 (worst) to 5 (best)**.
- Not every order has a completed delivery — `order_status` includes values such as `delivered`, `shipped`, `canceled`, and `unavailable`.
- ZIP code prefixes (not full postal codes) support city/state/geolocation analysis; multiple lat/long pairs can exist per prefix.
- The dataset spans **September 2016 – October 2018**, concentrated mainly in 2017–2018.
- All customer and seller identifiers are anonymized.

---

## 🎯 Key Analysis Areas

Revenue & Sales · Customer Analytics · Seller Performance · Logistics Performance · Payment Analysis · Customer Reviews · Geographic Insights

## 💼 Business Use Cases

Executive dashboards · Customer segmentation · Sales forecasting · Delivery optimization · Product quality analysis · Feature engineering for ML

---

## ⚠️ Known Limitations

- The dataset is historical (2016–2018) and does not reflect current marketplace conditions.
- Review text is in Portuguese and is not translated in this project (numeric `review_score` is used for sentiment classification instead).
- Geolocation data is at zip-prefix granularity, not exact address level.
- No product-level cost data is included, so only revenue — not profit/margin — can be analyzed directly.

---

## 🙏 Acknowledgement & License

Thanks to **Olist** for releasing this anonymized commercial dataset for educational and analytical purposes. The data is subject to the original [Olist dataset license on Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — refer to the Kaggle page for full terms before any commercial use.
