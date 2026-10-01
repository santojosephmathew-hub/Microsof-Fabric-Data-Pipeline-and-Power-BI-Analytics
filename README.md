
# Retail Data Engineering Project on Microsoft Fabric

An end-to-end data pipeline for a retail client using the **medallion architecture (Bronze → Silver → Gold)**. Raw files are stored in Azure Data Lake Storage, ingested with a Fabric Data Pipeline, cleaned with PySpark, aggregated into product KPIs, and visualised in Power BI.

## Business Requirements

- Build an end-to-end data pipeline for retail clients.
- Bring data from multiple sources into a data lake.
- Clean and standardise orders, inventory and returns data.
- Produce product-level KPIs and publish them to a Power BI dashboard.

## Architecture

```
Source files
    │
    ▼
Azure Data Lake Storage Gen2   (storage account: storagesanto, container: retail)
    │
    ▼
Fabric Data Pipeline           (Retail_pipeline – Copy data activities)
    │
    ▼
Lakehouse Files / Bronze       (Retail_LH1 – raw data)
    │
    ▼
PySpark Notebook               (Notebook_Retail – cleaning & transformation)
    │
    ▼
Silver Delta tables            (silver_orders, silver_inventory, silver_returns)
    │
    ▼
Gold Delta table               (gold_product_kpis)
    │
    ▼
Semantic model → Power BI      (retail_model → retail dashboard)
```

## Tech Stack

| Purpose | Tool |
|---|---|
| Raw storage | Azure Data Lake Storage Gen2 |
| Orchestration / ingestion | Microsoft Fabric Data Pipeline |
| Storage and tables | Fabric Lakehouse (Delta tables) |
| Transformation | Fabric Notebook, PySpark |
| Modelling and reporting | Power BI semantic model and report |

## Project Steps

### 1. Azure Data Lake Storage
Create a storage account (`storagesanto`) and a container named `retail`, then upload the raw source files.

![ADLS container](screenshots/01_adls_container.png)

### 2. Fabric Lakehouse
Create the workspace `retailWS` and the Lakehouse `Retail_LH1`. Under **Files**, organise the data into `Bronze`, `Silver` and `Gold` folders. Delta tables are created under `dbo`.

![Lakehouse](screenshots/02_lakehouse.png)

### 3. Ingestion pipeline (Bronze)
`Retail_pipeline` uses three **Copy data** activities (orders, inventory, returns) to copy the raw files into the Lakehouse Bronze area. The activities are chained and the run completes with all activities **Succeeded**.

![Pipeline](screenshots/03_pipeline.png)

### 4. Data cleaning with PySpark (Silver)
`Notebook_Retail` reads the raw files, for example:

```python
df_orders_raw = spark.read.csv(
    "Files/Bronze/orders_data.csv",
    header=True,
    inferSchema=True
)
display(df_orders_raw)
```

The raw orders data contains quality issues that are fixed in the Silver layer:

- Quantities written as words (`one`, `ONE`, `02`)
- Mixed date formats (`2023/06/12`, `12-07-2023`, `07-20-2023`, ...)
- Currency symbols and suffixes in amounts (`$`, `USD`, `INR`, `₹`)
- Spelling and case variations in delivery status (`delivrd`, `DELIVERD`, `pending`)
- Inconsistent customer IDs (`C001`, `C_002`) and NULL values

The cleaned data is saved as the Delta tables `silver_orders`, `silver_inventory` and `silver_returns`.

![Notebook](screenshots/04_notebook.png)

### 5. Gold layer
The Silver tables are joined and aggregated per product into `gold_product_kpis`, with these columns:

`ProductName`, `Total_Orders`, `Total_Quantity_Sold`, `Avg_Order_Value`, `Avg_Cost`, `Total_COGS`, `Net_Profit`, `Returned_Orders`, `Return_Rate_Percent`

### 6. Semantic model
The semantic model `retail_model` is created from `gold_product_kpis` and used as the source for the report.

![Semantic model](screenshots/05_semantic_model.png)

### 7. Power BI dashboard
The `retail dashboard` report shows revenue, net profit, returns and return rate by product, plus KPI cards for total revenue, total orders and net profit.

![Dashboard](screenshots/06_dashboard.png)

## Results

| KPI | Value |
|---|---|
| Total Revenue | 167.17K |
| Total Orders | 13 |
| Net Profit | -90.18K |

## Repository Structure

```
.
├── README.md
├── screenshots/
│   ├── 01_adls_container.png
│   ├── 02_lakehouse.png
│   ├── 03_pipeline.png
│   ├── 04_notebook.png
│   ├── 05_semantic_model.png
│   └── 06_dashboard.png
├── notebooks/        # add Notebook_Retail (.ipynb) here
└── docs/             # add the PDF project report here
```

## How to Run

1. Upload the raw source files to the `retail` container in ADLS.
2. Open `Retail_pipeline` in Fabric and click **Run** to load the Bronze data.
3. Run all cells in `Notebook_Retail` to build the Silver and Gold tables.
4. Refresh `retail_model`, then open the `retail dashboard` report.

## Future Improvements

- Verify the return-rate calculation and the product-name join (the dashboard currently shows 100% return rate for every product and a blank product).
- Add a date dimension and store dimension for time and location analysis.
- Schedule the pipeline with a trigger for automatic refresh.

## Author

Santo Joseph Mathew
