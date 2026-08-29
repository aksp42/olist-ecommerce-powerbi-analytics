# Olist Brazilian E-Commerce Public Dataset

Dataset used in the Olist E-Commerce Analytics Power BI project.

Overview:
The Olist Brazilian E-Commerce Public Dataset contains approximately 100,000 anonymized e-commerce orders placed between 2016 and 2018 across multiple Brazilian marketplaces.

Key Analysis Areas:
- Revenue and Sales
- Customer Analytics
- Seller Performance
- Logistics Performance
- Payment Analysis
- Customer Reviews
- Geographic Insights

Core Tables:
- olist_orders_dataset
- olist_order_items_dataset
- olist_order_payments_dataset
- olist_order_reviews_dataset
- olist_customers_dataset
- olist_sellers_dataset
- olist_products_dataset
- olist_geolocation_dataset
- product_category_name_translation

Primary Relationships:
Orders → Order Items (order_id)
Orders → Payments (order_id)
Orders → Reviews (order_id)
Orders → Customers (customer_id)
Order Items → Products (product_id)
Order Items → Sellers (seller_id)

Business Use Cases:
- Executive Dashboards
- Customer Segmentation
- Sales Forecasting
- Delivery Optimization
- Product Quality Analysis
- Feature Engineering

Data Characteristics:
- One order may contain multiple products.
- Multiple sellers can fulfill a single order.
- Review scores range from 1 to 5.
- ZIP codes support geolocation analysis.

Acknowledgement:
Thanks to Olist for releasing this anonymized commercial dataset for educational and analytical purposes.

License:
Subject to the original Olist Kaggle dataset license.
