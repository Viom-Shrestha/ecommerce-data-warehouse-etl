# E-Commerce Data Warehouse (SSIS ETL)

An ETL pipeline built in **SQL Server Integration Services (SSIS)** that loads raw e-commerce data (customers, orders, order items, products, reviews, suppliers) from flat files into a dimensional **star-schema data warehouse** (`ECommerce_DW`) in SQL Server, ready for BI/reporting and analytics.

Built as coursework for the Big Data module (BSc, 3rd year) — used here as a portfolio example of a full staging → dimension/fact ETL pipeline rather than a toy exercise.

## Architecture

```
Flat files (CSV)  →  Staging tables  →  Dimension & Fact loads  →  ECommerce_DW
```

**1. Staging** — each source file is loaded as-is into a staging table (`Customer_Staging`, `Order_Staging`, `OrderItem_Staging`, `Product_Staging`, `Review_Staging`, `Supplier_Staging`, `Date_Staging`), isolating raw extraction from transformation.

**2. Dimension loads** — one Data Flow Task per dimension:

| Task | Target | Notes |
|---|---|---|
| `Load_DimCustomer` | `DimCustomer` | Customer profile, location, segment, derived `age_group` |
| `Load_DimDate` | `DimDate` | Calendar attributes (weekday, month, quarter, year, weekend/holiday flags) |
| `Load_DimProduct` | `DimProduct` | Product, category, subcategory, pricing, supplier link |
| `Load_DimSupplier` | `DimSupplier` | Supplier details |
| `Load_DimShippingLocation` | `DimShippingLocation` | Shipping city/country derived from orders |
| `Load_DimOrder` | `DimOrder` | Order-level attributes (status, payment method, discount) |
| `Load_DimReview` | `DimReview` | Product review text, score, derived review length |

**3. Fact load** — `Load_FactSales` joins order items against the resolved dimension surrogate keys (via `Sort` + `Merge Join` stages), applies `Derived Column` and `Data Conversion` transforms, and routes rows through a `Conditional Split` that rejects records with missing keys or non-positive quantity/price before loading `FactSales` — so bad data is caught in the pipeline rather than the warehouse.

## Tech stack

- **SSIS** (Visual Studio / SQL Server Data Tools project, deployment model: Project)
- **SQL Server** as the target data warehouse, connected via Windows Integrated Security
- Flat File sources (CSV) for all extracts

## Project structure

```
ECommerceDW.slnx              solution file
ECommerceDW/
  ECommerceDW.dtproj          SSIS project file
  ECommerceDW.database        data warehouse database reference
  Viom.dtsx                   main ETL package (staging → dimensions → fact)
  Flat File Connection Manager.conmgr
data/                         place source CSVs here (not included — see below)
```

## Running it

1. Open `ECommerceDW.slnx` in Visual Studio with the SSIS (SQL Server Integration Services) extension installed.
2. Create an `ECommerce_DW` database on a local SQL Server instance and update the connection manager if your instance name differs from `localhost\SQLEXPRESS`.
3. Place the source CSVs in `data/` at the repo root, named: `customers.csv`, `orders.csv`, `order_items.csv`, `products.csv`, `reviews.csv`, `suppliers.csv`, `date_dim_helper.csv`.
4. Execute the `Viom` package.

The source dataset isn't included in this repo (it was supplied as part of the coursework) — bring your own CSVs matching the column names visible in the connection managers, or adapt the flat file sources to your own schema.
