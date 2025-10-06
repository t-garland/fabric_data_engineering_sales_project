# Musical Magic Instruments - Data Engineering Project

## Project Overview
End-to-end data engineering solution for a fictional musical instruments retailer, implementing medallion architecture in Microsoft Fabric with PySpark transformations and Power BI semantic models.

## Architecture

### Medallion Lakehouse Design
Bronze Layer (Raw Data)
* customers.csv
* products.csv
* orders_header.csv
* order_lines.csv
* reviews.json
* social_media.json
* web_logs.json

Silver Layer (Cleaned & Validated)
* customers_silver (SCD Type 2)
* products_silver (SCD Type 2)
* orders_header_silver
* order_lines_silver
* reviews_silver
* social_media_silver
* web_logs_silver

Gold Layer (Analytics-Ready)
* dim_date_gol
* dim_customer_gold
* dim_product_gold
* fact_sales_gold
* fact_reviews
* fact_web_events

Semantic Models
* Sales Analytics
* Customer Journey Analytics

## Technologies Used
- **Microsoft Fabric**: Cloud data platform
- **PySpark**: Data transformation and processing
- **Delta Lake**: ACID transactions and time travel
- **Power BI**: Business intelligence and reporting
- **Python**: Orchestration and automation

## Data Model

### Star Schema - Sales Analytics
- **Fact Table**: fact_sales_gold
- **Dimensions**: dim_customer_gold, dim_product_gold, dim_date_gold
- **Grain**: One row per order line item
- **Measures**: Revenue, profit, quantity, margins

### Constellation Schema - Customer Journey Analytics
- **Fact Tables**: fact_reviews, fact_web_events
- **Dimensions**: dim_customer_gold, dim_product_gold, dim_date_gold
- **Use Cases**: Customer behavior analysis, review sentiment, web conversion funnels

## Key Technical Decisions

### SCD Implementation
- **SCD Type 2**: Customers and products (track historical changes)
- **SCD Type 1**: Gold dimensions (current state only for analytics)
- **Append-Only**: Immutable events (reviews, web logs, transactions)

### Idempotency Pattern
All transformations use Delta Lake merge operations to ensure pipeline reruns don't create duplicates:
- Composite keys for event data (customer_id + timestamp)
- Primary keys for transactional data (order_id)
- Row hashing for SCD2 change detection

### Data Quality
- Schema enforcement at Bronze → Silver
- NULL handling and default values
- Business rule validation (rating 1-5, valid dates)
- Deduplication using dropDuplicates()

## Pipeline Orchestration

**Pipeline 1: Data Extraction**
* Ingests raw files to Bronze layer

**Pipeline 2: Transform & Serve**
1. Silver transformations (data quality, standardization)
2. Gold transformations (dimensional modeling)
3. Semantic model refresh (Sales + Customer Journey)

**Scheduling**: Daily execution with success dependencies

## Business Value

### Sales Analytics
- Revenue and profit tracking by customer/product/time
- Margin analysis and discount impact
- Payment method preferences
- Order volume trends

### Customer Journey Analytics
- Web browsing to purchase conversion
- Product review sentiment analysis
- Customer engagement patterns
- Page performance metrics

## Challenges & Solutions

**Challenge**: Timestamp display showing "Invalid Date" in SQL endpoint  
**Solution**: Explicit TimestampType() casting and Fabric-specific configurations

**Challenge**: SCD2 implementation complexity  
**Solution**: Two-step merge pattern (close old records, insert new versions) using row hashing

**Challenge**: Social media data lacks dimensional relationships  
**Solution**: Kept in Silver layer for standalone analysis, excluded from Gold star schema

## Project Structure
/notebooks
* Silver_Transformations.ipynb
* Gold_Transformations.ipynb
/pipelines
* Data_Extraction_Pipeline.json
* Transform_And_Serve_Pipeline.json
/semantic_models
* Sales_Analytics.pbix
* Customer_Journey_Analytics.pbix
/documentation
* Architecture_Diagram.png

## Running the Project

### Prerequisites
* Microsoft Fabric workspace with trial/premium capacity
* Lakehouse created (Bronze, Silver, Gold)
* Sample data files uploaded to Bronze layer

### Setup Steps
1. Create Fabric workspace with lakehouse
2. Upload sample data to Bronze layer folders
3. Import notebooks to workspace
4. Run Silver_Transformations notebook
5. Run Gold_Transformations notebook
6. Create semantic models in Power BI
7. Configure pipeline orchestration
8. Schedule daily refresh

## Data Sources
Sample datasets generated using Python Faker library to simulate:
* Customer records (10 records)
* Product catalog (musical instruments)
* Sales transactions (orders and line items)
* Customer reviews (JSON format)
* Social media mentions (JSON format)
* Website clickstream data (JSON format)

## Contact
* Tristan Garland
* [LinkedIn](https://www.linkedin.com/in/tristan-garland-78a75aba/)
* [Email](t.garland@me.com)

## License
This project is for portfolio demonstration purposes.
