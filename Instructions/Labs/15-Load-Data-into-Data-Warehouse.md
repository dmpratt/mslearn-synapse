---
lab:
    title: 'Load Data into a Relational Data Warehouse'
    module: 'Loading Azure Data Lake data into Azure Synapse Analytics'
---

# Load Data into a Relational Data Warehouse

In this lab, we're going to load data into a dedicated SQL Pool that is a way to gain optimal performance of large datasets and warehouses. We'll use the Create Table as SELECT (CTAS) operation to move data into the dedicated SQL Pool.

In this lab, you'll use a dedicated SQL pool in Azure Synapse Analytics to transform data in files into physical tables in Azure Synapse Analytics.

This lab will take approximately **30** minutes to complete.

## Before you start

You'll need an [Azure subscription](https://azure.microsoft.com/free) in which you have administrative-level access.

## Provision an Azure Synapse Analytics workspace

You'll need an Azure Synapse Analytics workspace with access to data lake storage. You can use the built-in serverless SQL pool to query files in the data lake.

In this exercise, you'll use a combination of a PowerShell script and an ARM template to provision an Azure Synapse Analytics workspace.

1. Sign into the [Azure portal](https://portal.azure.com) at `https://portal.azure.com`.
2. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new Cloud Shell in the Azure portal, selecting a ***PowerShell*** environment and creating storage if prompted. The Cloud Shell provides a command line interface in a pane at the bottom of the Azure portal, as shown here:

    ![Azure portal with a cloud shell pane](./images/cloud-shell.png)

    > **Note**: If you have previously created a cloud shell that uses a *Bash* environment, use the the drop-down menu at the top left of the cloud shell pane to change it to ***PowerShell***.

3. Note that the Cloud Shell can be resized by dragging the separator bar at the top of the pane, or by using the **&#8212;**, **&#9723;**, and **X** icons at the top right of the pane to minimize, maximize, and close the pane. For more information about using the Azure Cloud Shell, see the [Azure Cloud Shell documentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. In the PowerShell pane, enter the following commands to clone this repo:

    ```powershell
    rm -r dp-000 -f
    git clone https://github.com/MicrosoftLearning/mslearn-synapse dp-000
    ```

5. After the repository has been cloned, enter the following commands to change to the folder for this lab and run the **setup.ps1** script it contains:

    ```powershell
    cd dp-000/Allfiles/Labs/15
    ./setup.ps1
    ```

6. If prompted, choose which subscription you want to use (this will only happen if you have access to multiple Azure subscriptions).
7. When prompted, enter a suitable password to be set for your Azure Synapse SQL pool.

    > **Note**: Be sure to remember this password!

8. Wait for the script to complete - this typically takes around 10 minutes, but in some cases may take longer. While you are waiting, review the [Apache Spark Pool Configurations](https://learn.microsoft.com/en-us/azure/synapse-analytics/spark/apache-spark-pool-configurations) article in the Azure Synapse Analytics documentation.

## View and Navigate Synapse Worspace
1. After the script has completed, in the Azure portal, go to the dp000-xxxxxxx resource group that it created, and select your Synapse workspace.
2. In the Overview page for your Synapse Workspace, in the Open Synapse Studio card, select Open to open Synapse Studio in a new browser tab; signing in if prompted.
3. On the left side of Synapse Studio, use the ›› icon to expand the menu - this reveals the different pages within Synapse Studio that you’ll use to manage resources and perform data analytics tasks.
4. On the Data page, view the Linked tab and verify that your workspace includes a link to your Azure Data Lake Storage Gen2 storage account, which should have a name similar to **synapsexxxxxxx (Primary - datalakexxxxxxx)**.
5. Expand your storage account and verify that it contains a file system container named **files (primary)**.
6. Select the files container, and note that it contains folders named data and synapse. The synapse folder is used by Azure Synapse, and the data folder contains the data files you are going to query.
Open the sales folder and the orders folder it contains, and observe that the orders folder contains .csv files for dimCustomer, dimDate, dimProduct, and FactInternetSales data.
***Right-click*** any of the files and select Preview to see the data it contains. Note that the files contain a header row, so you can select the option to display column headers.

## Load data warehouse tables

One of the most common patterns for loading a data warehouse is to transfer data from source systems to files in a data lake, ingest the file data into staging tables, and then use SQL statements to load the data from the staging tables into the dimension and fact tables. Usually data loading is performed as a periodic batch process in which inserts and updates to the data warehouse are coordinated to occur at a regular interval (for example, daily, weekly, or monthly).

There are many technologies you can use to load data, including pipelines created using Azure Synapse Analytics or Azure Data Factory, SQL Server Integration Services packages, or command line tools like the bulk copy program (BCP). In this unit, we'll focus on SQL-based techniques to ingest data from a data lake.

## Loading data into staging tables

If you use external tables for staging, there's no need to load the data into them because they already reference the data files in the data lake. However, if you use "regular" relational tables, you can use the COPY statement to load data from the data lake, as shown in the following example:

>**NOTE**: Change the ***mydatalake*** with the name of your datalake name created during the beginning of the lab

```sql
COPY INTO dbo.StageProduct
    (ProductID, ProductAlternateKey, ProductName, ProductCategory, Color, Size, ListPrice, Discontinued)
FROM 'https://mydatalake.blob.core.windows.net/files/data/StageProducts.csv'
WITH
(
    FILE_TYPE = 'CSV',
    MAXERRORS = 0
    IDENTITY_INSERT = 'OFF',
    FIRSTROW = 2
);
```

## Using a CREATE TABLE AS (CTAS) statement

> ***NOTE*** For more Information, see [CREATE TABLE AS SELECT (CTAS)](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-develop-ctas) in the Azure Synapse Analytics documention.

For example, the following code creates a new DimProduct table based on the results of a query that retrieves data from the StageProduct table:

```sql
CREATE TABLE dbo.DimProduct
WITH
(
    DISTRIBUTION = REPLICATE,
    CLUSTERED COLUMNSTORE INDEX
)
AS
SELECT ROW_NUMBER() OVER(ORDER BY ProductID) AS ProductKey,
       ProductID AS ProductAltKey,
       ProductName,
       ProductCategory,
       Color,
       Size,
       ListPrice,
       Discontinued
FROM dbo.StageProduct;
```

By default, tables are Round Robin distributed. This default makes it easy for users to start creating tables without having to decide how their tables should be distributed. Round Robin tables may perform sufficiently for some workloads. But, in most cases, a distribution column provides better performance.

The most common example of a table distributed by a column outperforming a round robin table is when two large fact tables are joined.

For example, if you have an orders table distributed by order_id, and a transactions table also distributed by order_id, when you join your orders table to your transactions table on order_id, this query becomes a pass-through query. Data movement operations are then eliminated. Fewer steps mean a faster query. Less data movement also makes for faster queries.

The CTAS operation will allow us to use the round-robin table type for loading in those cases and then create a distributed table once the data is understood within the warehouse.

## Using a CREATE EXTERNAL TABLE AS SELECT (CETAS)

```sql
-- Create the external datasource (changing the suffix to match yours
IF NOT EXISTS (SELECT * FROM sys.external_data_sources WHERE name = 'MyDataSource') 
 CREATE EXTERNAL DATA SOURCE [MyDataSource] 
 WITH (
  LOCATION = 'abfss://files@datalakerpqavis.dfs.core.windows.net', 
  TYPE = HADOOP 
 )

 -- Create the parquet format with a Gzip Codec
CREATE EXTERNAL FILE FORMAT MyParquet  
WITH (  
    FORMAT_TYPE = PARQUET  
    , DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec'  
 );  

-- Create the external table
CREATE EXTERNAL TABLE hdfsCustomer  
WITH (  
    LOCATION='/Data/customer.tbl',  
 DATA_SOURCE = MyDataSource, --The Data Source created above
 FILE_FORMAT = MyParquet --The File Format created above
) AS SELECT * FROM dimCustomer;  

select top 100 * from hdfsCustomer
```

## Optimize Load Performance

After loading new data into the data warehouse, it's a good idea to rebuild the table columnstore indexes and update statistics on commonly queried columns.

The following example rebuilds all indexes on the DimProduct table.

```sql
ALTER INDEX ALL ON dbo.DimProduct REBUILD
```

The following example creates statistics on the ProductCategory column of the DimProduct table:

```sql
CREATE STATISTICS productcategory_stats
ON dbo.DimProduct (ProductCategory);
```

>***NOTE*** For more information, see the [Indexes on dedicated SQL pool tables in Azure Synapse Analytics](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-index) and [Table statistics for dedicated SQL pool in Azure Synapse Analytics articles](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-statistics) in the Azure Synapse Analytics documentation.

## Delete Azure resources

If you've finished exploring Azure Synapse Analytics, you should delete the resources you've created to avoid unnecessary Azure costs.

1. Close the Synapse Studio browser tab and return to the Azure portal.
2. On the Azure portal, on the **Home** page, select **Resource groups**.
3. Select the **dp000-*xxxxxxx*** resource group for your Synapse Analytics workspace (not the managed resource group), and verify that it contains the Synapse workspace, storage account, and Spark pool for your workspace.
4. At the top of the **Overview** page for your resource group, select **Delete resource group**.
5. Enter the **dp000-*xxxxxxx*** resource group name to confirm you want to delete it, and select **Delete**.

    After a few minutes, your Azure Synapse workspace resource group and the managed workspace resource group associated with it will be deleted.
