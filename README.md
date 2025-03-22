# DataWarehouse End To End

![dbt](https://github.com/user-attachments/assets/31d4e0a9-5ed9-4b82-95e9-874b0840cb51)



## Table of Contents 
- [Introduction](#introduction)
- [Tech Stack & Tools](#tech-stack--tools)
- [Pipeline Architecture](#pipeline-architecture)
- [DBT Models](#DBT-Models)
- [Snowflake](#data-warehouse-Snowflake)
- [Reporting](#reporting)

![workflow](https://github.com/user-attachments/assets/10f5b452-f6cb-4714-85e3-01c311359096)



## Introduction 
This project designed to ingest and transform data from multiple sources (CRM and ERP systems) into Snowflake. It follows the Medallion Architecture to structure data efficiently for analytics. The pipeline leverages dbt (Data Build Tool) to transform raw data into analytics-ready datasets, ensuring high-quality, governed, and optimized data models for reporting and business intelligence.



## Tech Stack & Tools
- **DBT (Data Build Tool)**: For building and transforming data models.
- **Snowflake**: As the data warehouse.
- **Tableau** : For visualizing the reporting layer.

## Pipeline Architecture 

[end to end data warehouse project using dbt.pdf](https://github.com/user-attachments/files/19399620/end.to.end.data.warehouse.project.using.dbt.pdf)



The project follows the Medallion Architecture, which organizes data into three layers:

    Bronze Layer (Raw Data): Stores unprocessed and ingested data from various sources.
    Silver Layer (Cleansed Data): Cleans and pre-processes data for transformation and enrichment.
    Gold Layer (Aggregated Data): Optimized for analytics, reporting, and business intelligence.




## DBT Models
#### customer_cte
 
    {{
    config(
        materialized='incremental',
        unique_key='ID',
        indexes=[{"columns": ['ID'], "unique": true}],
        target_schema='silver'
    )
    }}
    
    with customer_cte as (
        SELECT 
            *, 
            row_number() OVER (PARTITION BY cst_id ORDER BY cst_create_date DESC) AS last_update
        FROM {{ source('row_data', 'crm_cust_info') }}
    )
    
    SELECT 
        cst_id AS ID,
        cst_key AS customer_key, 
        TRIM(cst_firstname) AS FIRST_NAME, 
        TRIM(cst_lastname) AS LAST_NAME,
        CASE 
            WHEN UPPER(cst_marital_status) = 'S' THEN 'Single'
            WHEN UPPER(cst_marital_status) = 'M' THEN 'Married'
            ELSE 'n/a'
        END AS MARITAL_STATUS,
        CASE 
            WHEN UPPER(cst_gndr) = 'F' THEN 'Female'
            WHEN UPPER(cst_gndr) = 'M' THEN 'Male'
            ELSE 'n/a'
        END AS gender,
        cst_create_date 
    FROM customer_cte
    WHERE last_update = 1 and cst_id is not null
#### sales_cte

    with sales_cte as (
        SELECT 
            sls_ord_num AS order_number,
            sls_prd_key AS product_key,
            sls_cust_id AS Customer_id,
            CASE 
                WHEN sls_order_dt = 0 OR LENGTH(sls_order_dt) != 8 THEN TO_DATE(SLS_SHIP_DT::VARCHAR, 'YYYYMMDD') - INTERVAL '2 DAY'
                ELSE TO_DATE(sls_order_dt::VARCHAR, 'YYYYMMDD') 
            END AS order_date,
            CASE 
                WHEN SLS_SHIP_DT = 0 or length(SLS_SHIP_DT) != 8 then NULL
                ELSE TO_DATE(SLS_SHIP_DT::VARCHAR,'YYYYMMDD')
            end as Ship_date,
            CASE 
                WHEN SLS_DUE_DT = 0 or length(SLS_DUE_DT) != 8 then NULL
                else to_date(SLS_DUE_DT::VARCHAR,'YYYYMMDD')
            end as DUE_Date,
            CASE 
                When sls_sales is null or sls_sales <= 0 or sls_sales != sls_quantity * abs(sls_price) then sls_quantity * abs(sls_price)
                else sls_sales
            end as sales,
            sls_quantity as quantity ,
            CASE 
                when sls_price is null or sls_price <= 0 then sls_sales / nullif(sls_quantity,0) 
                else sls_price
            end as price
        FROM {{ source('row_data', 'crm_sales_details') }} s
    )
    SELECT * from sales_cte



## Data Warehouse Snowflake


  ![snwoflake](https://github.com/user-attachments/assets/ca7f2fc1-8600-448c-b7d8-1612282bcc0b)


  






## Reporting

![dash_1](https://github.com/user-attachments/assets/83ac34b0-2e8c-4893-bee5-c89cf6f554f8)
![dash_2](https://github.com/user-attachments/assets/5c2d51d4-230e-41cd-a5d8-737be7fce5a9)
![Data_Model_Tableau](https://github.com/user-attachments/assets/a7d31c72-f8ef-472b-912e-f0d7104845b6)





# Contact Information
📧 Email: [mhmwdalarf97@gmail.com)  
🔗 LinkedIn: Abdelnaem Alaref






