# 🚀 SQL Data Warehouse

**The Ultimate PostgreSQL Data Warehouse Solution**

A comprehensive, enterprise-grade data warehouse implementation using PostgreSQL, featuring well-structured ETL
pipelines, clear data models, and integrated analytics for end-to-end data
management. Created by AMIR KHAN.

## 🚀 What is SQL Data Warehouse?

SQL Data Warehouse is a cutting-edge data warehouse solution that transforms raw business data into actionable insights. Built with PostgreSQL at its core, it follows a sophisticated layered architecture (Bronze, Silver, Gold) to seamlessly manage data from CRM and ERP sources. 

The platform handles customer, product, and sales data with precision, performing advanced ETL operations to clean, transform, and aggregate information for powerful business analytics and decision-making.

The Bronze layer stores raw data loaded from CSV files. The Silver layer cleans
and standardizes this data using PL/pgSQL procedures, handling tasks like
deduplication, trimming, and value standardization. The Gold layer provides
curated views using dimensional modeling (star schema), reconciling data from
multiple sources to create unified customer and product profiles along with
sales facts. This setup supports reporting, dashboards, and analytical queries,
ensuring data consistency through business rules.

Key problems solved:

- Integrating disparate data from CRM and ERP systems.
- Ensuring data quality through cleaning and reconciliation.
- Providing optimized structures for business intelligence tools like BI
  reporting, SQL queries, ad-hoc analysis, and machine learning.

## ⚡ Key Features

- **🔥 Layered Architecture**: Bronze (raw), Silver (cleaned), Gold (curated)
  layers for progressive data refinement and maximum performance
- **🚀 Advanced ETL Pipelines**: PL/pgSQL procedures for loading, cleaning, and
  transforming data, with comprehensive timing logs for performance monitoring
- **🎯 Smart Data Reconciliation**: Intelligently merges CRM and ERP data, preferring sources based on
  sophisticated business rules (e.g., CRM for gender unless unknown, then ERP)
- **📊 Dimensional Modeling**: Star schema in the Gold layer with optimized dimension
  tables (customers, products) and fact table (sales) for lightning-fast queries
- **⚙️ One-Click Setup**: `init_datawarehouse.sql` for full initialization and
  `data_refresh.sql` for seamless data updates without downtime
- **🔒 Enterprise Security**: Restricts public access to the database schema with advanced security controls
- **📚 Complete Documentation**: Includes data flow diagrams, architecture overview, data
  models, and comprehensive data catalog for the Gold layer

## Architecture

The data warehouse follows a medallion architecture:

- **Bronze Layer**: Raw data ingestion from sources. Tables mirror source
  structures with minimal transformation (full load/truncate-insert from CSVs).
- **Silver Layer**: Data cleaning and standardization. Uses temporary tables
  for processing, removes nulls/duplicates, trims strings, standardizes
  values (e.g., 'M' to 'Married'), and adds timestamps.
- **Gold Layer**: Business-ready data. Views aggregate and join Silver data,
  applying business logic for reconciliation and using surrogate keys for
  dimensions.

Consumers can access the Gold layer via BI tools, SQL queries, ad-hoc analysis,
or machine learning interfaces.

![Data Warehouse Architecture](documents/DataWarehouseArchitecture.jpg)

## Data Flow

Data flows from sources (CRM and ERP CSVs) into Bronze tables, then transformed
into Silver, and finally aggregated into Gold views.

- **Sources**: CRM (customer info, product info, sales details) and ERP (
  customer demographics, locations, product categories).
- **Bronze**: Direct copies like `bronze.crm_cust_info`,
  `bronze.erp_cust_az12`.
- **Silver**: Cleaned versions like `silver.crm_cust_info`, with procedures for
  each table.
- **Gold**: Views like `gold.dim_customers`, `gold.dim_products`,
  `gold.fact_sales`.

![Data Flow Diagram](documents/DataFlowDiagram.jpg)
![Data Integration Diagram](documents/DataEDA.jpg)

## Data Model

The Gold layer uses a star schema for efficient querying:

- **Fact Table**: `gold.fact_sales` – Contains measures like sales_amount,
  quantity, price, with foreign keys to dimensions.
- **Dimension Tables**:
    - `gold.dim_customers` – Customer details with surrogate key.
    - `gold.dim_products` – Product details with surrogate key.

![Gold Layer Data Model](documents/GoldLayerDataModel.jpg)

## Data Catalog

The Gold layer contains business-level representations of data, designed to
support business-level analytics. It aggregates and reconciles data from the
Silver layer, providing unified customer and product profiles along with sales
fact data. These views are optimized for reporting, dashboards, and analytical
queries, leveraging surrogate keys for dimensional modeling and ensuring data
consistency through predefined business rules.

### 1. gold.dim_customers

| Column Name     | Data Type | Description                                                                                                   | Example Value    |
|-----------------|-----------|---------------------------------------------------------------------------------------------------------------|------------------|
| customer_number | INTEGER   | A unique surrogate key assigned sequentially based on cst_id, may shift on data changes.                      | 1                |
| customer_id     | VARCHAR   | Unique identifier for the customer from CRM data.                                                             | 11000            |
| customer_key    | VARCHAR   | Alternate key for the customer, used for cross-system matching.                                               | AW00011000       |
| first_name      | VARCHAR   | Customer's first name from CRM.                                                                               | John             |
| last_name       | VARCHAR   | Customer's last name from CRM.                                                                                | Doe              |
| country         | VARCHAR   | Customer's country from ERP location data, defaults to 'Unknown' if missing.                                  | Australia        |
| gender          | VARCHAR   | Customer's gender, prefers CRM value unless 'Unknown', then uses ERP gen if available, defaults to 'Unknown'. | Male             |
| marital_status  | VARCHAR   | Customer's marital status from CRM.                                                                           | Married / Single |
| birth_date      | DATE      | Customer's birth date from ERP, falls back to create_date if missing.                                         | 05-15-1990       |
| create_date     | DATE      | Date when the customer record was created in CRM.                                                             | 01-10-2023       |

### 2. gold.dim_products

(Based on the provided gold_ddl.sql script; add actual examples if available)

| Column Name    | Data Type | Description                                                                    | Example Value  |
|----------------|-----------|--------------------------------------------------------------------------------|----------------|
| product_number | INTEGER   | A unique surrogate key assigned sequentially based on prd_start_dt and prd_id. | 1              |
| product_id     | INTEGER   | Unique identifier for the product from CRM data.                               | 123            |
| product_key    | VARCHAR   | Alternate key for the product.                                                 | BK-M68B-42     |
| product_name   | VARCHAR   | Product name from CRM.                                                         | Mountain-200   |
| category_id    | VARCHAR   | Category ID from CRM.                                                          | 1              |
| category       | VARCHAR   | Product category from ERP.                                                     | Bikes          |
| sub_category   | VARCHAR   | Product sub-category from ERP.                                                 | Mountain Bikes |
| maintenance    | VARCHAR   | Maintenance flag from ERP (Yes/No/Unknown).                                    | Yes            |
| cost           | NUMERIC   | Product cost from CRM.                                                         | 1250.00        |
| product_line   | VARCHAR   | Product line from CRM.                                                         | M              |
| start_date     | DATE      | Product start date from CRM.                                                   | 2011-07-01     |
| end_date       | DATE      | Product end date from CRM.                                                     | NULL           |

### 3. gold.fact_sales

| Column Name     | Data Type | Description                                   | Example Value |
|-----------------|-----------|-----------------------------------------------|---------------|
| order_number    | VARCHAR   | Sales order number from CRM.                  | SO43659       |
| product_number  | INTEGER   | Surrogate key referencing gold.dim_products.  | 1             |
| customer_number | INTEGER   | Surrogate key referencing gold.dim_customers. | 1             |
| order_date      | DATE      | Order date from CRM.                          | 2011-05-31    |
| shipping_date   | DATE      | Shipping date from CRM.                       | 2011-06-07    |
| due_date        | DATE      | Due date from CRM.                            | 2011-06-12    |
| sales_amount    | NUMERIC   | Total sales amount.                           | 2319.99       |
| quantity        | INTEGER   | Quantity sold.                                | 1             |
| price           | NUMERIC   | Unit price.                                   | 2319.99       |

## Project Structure

- `/scripts/`: Contains all SQL scripts.
    - `init_datawarehouse.sql`: Master script for initializing the entire
      warehouse.
    - `data_refresh.sql`: Script for refreshing data.
    - `/bronze/`: Bronze DDL and load scripts.
    - `/silver/`: Silver DDL and load procedures.
    - `/gold/`: Gold view creation script.
- `/docs/`: Diagrams and data catalog (PDFs like DataFlowDiagram.pdf,
  DataCatalog.pdf).
- Source data CSVs are referenced in load scripts (e.g., under
  `project_files/data_warehouse_project/source_crm/`).

## Prerequisites

To set up and run this data warehouse project, you'll need the following software:

- **PostgreSQL**: The database server (version 17 or compatible). Download and install from the official PostgreSQL website: [https://www.postgresql.org/download/](https://www.postgresql.org/download/).
- **psql Client**: The command-line tool for interacting with PostgreSQL (included with PostgreSQL installation).
- **Database Client**: A GUI tool for querying and managing the database, such as pgAdmin (included with PostgreSQL), DataGrip, or VS Code with the PostgreSQL extension.

Ensure PostgreSQL is running on your system, and you have administrative access to create databases and schemas.

You'll also need the source CSV datasets (e.g., `cust_info.csv`, `prd_info.csv`, etc., from CRM and ERP sources). These are referenced in the load scripts and must be placed in a specific directory accessible by PostgreSQL.

## Installation and Setup

Follow these steps to install and configure the data warehouse. The process involves placing CSV files, updating file paths in scripts, and executing master scripts via psql.

### Step 1: Dataset CSV Files
The project includes sample CSV files in the `datasets/` directory. The scripts are configured to use relative paths, so ensure you run the SQL scripts from the project root directory.

The CSV files are already organized in the following structure:
```
datasets/
├── source_crm/
│   ├── cust_info.csv
│   ├── prd_info.csv
│   └── sales_details.csv
└── source_erp/
    ├── CUST_AZ12.csv
    ├── LOC_A101.csv
    └── PX_CAT_G1V2.csv
```

### Step 2: File Paths Configuration
The scripts are now configured to use relative paths from the project root directory. No manual path updates are required. The `bronze_load.sql` script will automatically find the CSV files in the `datasets/` directory.

### Step 3: Script Paths Configuration
The `init_datawarehouse.sql` script is configured to use relative paths. Ensure you run the script from the project root directory where the `scripts/` folder is located.

### Step 4: Data Refresh Configuration
The `data_refresh.sql` script is also configured with relative paths. No manual updates are required.

### Step 5: Run the Initialization Script
Execute `init_datawarehouse.sql` to set up the database. This script drops and recreates the `sql_data_warehouse` database, sets up schemas, loads data, and builds layers.

**Important**: Run the script from the project root directory (where the `scripts/` folder is located).

Use one of these methods:

- **Option 1: From Terminal/Command Prompt**  
  Navigate to the project root directory and run:  
  ```bash
  psql -h localhost -U your_username -d postgres -f scripts/init_datawarehouse.sql
  ```  
  Replace `localhost` with your host (if not local) and `your_username` with your PostgreSQL username. You'll be prompted for your password.

- **Option 2: Inside psql**  
  Open psql (e.g., `psql -U your_username -d postgres`), then run:  
  ```sql
  \i 'scripts/init_datawarehouse.sql'
  ```

If everything is configured correctly (paths, permissions, and CSVs in place), the warehouse will set up in seconds, and you'll see a success message: "Data Warehouse created and initialized successfully!"

### Step 6: Refresh Data (Optional)
If you update the source CSVs later, run `data_refresh.sql` to reload Bronze/Silver and rebuild Gold without recreating the database.

**Important**: Run from the project root directory.

Use similar methods as Step 5, but connect to `sql_data_warehouse` instead of `postgres` for the terminal option:  
```bash
psql -h localhost -U your_username -d sql_data_warehouse -f scripts/data_refresh.sql
```

Or inside psql (after `\c sql_data_warehouse`):  
```sql
\i 'scripts/data_refresh.sql'
```

You'll see: "Data Refreshed Successfully!"

### Troubleshooting
- **Permission Issues**: Ensure the PostgreSQL service account has read access to your CSV directory. On Windows, this is usually the `postgres` user.
- **Path Errors**: Double-check double backslashes (`\\`) in Windows paths within SQL scripts.
- **Connection Errors**: Verify PostgreSQL is running and your username/password are correct.
- After setup, connect via your database client (e.g., pgAdmin) to `sql_data_warehouse` and query the Gold layer views (e.g., `SELECT * FROM gold.dim_customers LIMIT 10;`) to verify.

## Author

**AMIR KHAN** - Data Engineer and SQL Developer

This project demonstrates expertise in:
- PostgreSQL database design and optimization
- ETL pipeline development using PL/pgSQL
- Data warehouse architecture (Bronze, Silver, Gold layers)
- Dimensional modeling and star schema design
- Data quality and reconciliation processes

## Future Improvements

Planned enhancements include:
- Metadata and audit schema for data quality monitoring
- Data flow logging and performance metrics
- Automated data validation and error handling
- Enhanced documentation and API interfaces