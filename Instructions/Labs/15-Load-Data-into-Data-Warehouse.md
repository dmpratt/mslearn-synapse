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

3. Note that Cloud Shell can be resized by dragging the separator bar at the top of the pane, or by using the—, **&#9723;**, and **X** icons at the top right of the pane to minimize, maximize, and close the pane. For more information about using the Azure Cloud Shell, see the [Azure Cloud Shell documentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. In the PowerShell pane, enter the following commands to clone this repository:

    ```powershell
    rm -r dp-000 -f
    git clone https://github.com/MicrosoftLearning/mslearn-synapse dp-000
    ```

5. After the repository has been cloned, enter the following commands to change to the folder for this lab, and run the **setup.ps1** script it contains:

    ```powershell
    cd dp-000/Allfiles/Labs/15
    ./setup.ps1
    ```

6. If prompted, choose which subscription you want to use (this will only happen if you have access to multiple Azure subscriptions).
7. When prompted, enter a suitable password to be set for your Azure Synapse SQL pool.

    > **Note**: Be sure to remember this password!

8. Wait for the script to complete - this typically takes around 10 minutes, but in some cases may take longer. While you're waiting, review the [dedicated SQL pool (formerly SQL DW) in Azure Synapse Analytics](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is) article in the Azure Synapse Analytics documentation.

## View and Navigate Synapse Workspace

1. After the script has completed, in the Azure portal, go to the dp000-xxxxxxx resource group that it created, and select your Synapse workspace.
2. In the Overview page for your Synapse Workspace, in the Open Synapse Studio card, select Open to open Synapse Studio in a new browser tab; signing in if prompted.
3. On the left side of Synapse Studio, use the ›› icon to expand the menu - this reveals the different pages within Synapse Studio that you’ll use to manage resources and perform data analytics tasks.
4. On the Data page, view the Linked tab and verify that your workspace includes a link to your Azure Data Lake Storage Gen2 storage account, which should have a name similar to **synapsexxxxxxx (Primary - datalakexxxxxxx)**.
5. Expand your storage account and verify that it contains a file system container named **files (primary)**.
6. Select the files container, and note that it contains folders named data and synapse. The synapse folder is used by Azure Synapse, and the data folder contains the data files you're going to query.
Open the sales folder and the orders folder it contains, and observe that the orders folder contains .csv files for dimCustomer, dimDate, dimProduct, and FactInternetSales data.
***Right-click*** any of the files and select Preview to see the data it contains. Note the files contain a header row, so you can select the option to display column headers.

### Start the dedicated SQL pool

1. In the Azure portal, close the query editor for your Azure SQL database (discarding any changes) and return to the page for your **dp000-*xxxxxxx*** resource group.
2. Open the **synapse*xxxxxxx*** Synapse workspace, and on its **Overview** page, in the **Open Synapse Studio** card, select **Open** to open Synapse Studio in a new browser tab; signing in if prompted.
3. On the left side of Synapse Studio, use the **&rsaquo;&rsaquo;** icon to expand the menu - this reveals the different pages within Synapse Studio.
4. On the **Manage** page, on the **SQL pools** tab, select the row for the **sql*xxxxxxx*** dedicated SQL pool and use its **&#9655;** icon to start it; confirming that you want to resume it when prompted.
5. Wait for the SQL pool to resume. This can take a few minutes. You can use the **&#8635; Refresh** button to check its status periodically. The status will show as **Online** when it is ready.

## Load data warehouse tables

One of the most common patterns for loading a data warehouse is to transfer data from source systems to files in a data lake, ingest the file data into staging tables, and then use SQL statements to load the data from the staging tables into the dimension and fact tables. Usually data loading is performed as a periodic batch process in which inserts and updates to the data warehouse are coordinated to occur at a regular interval (for example, daily, weekly, or monthly).

There are many technologies you can use to load data, including pipelines created using Azure Synapse Analytics or Azure Data Factory, SQL Server Integration Services packages, or command line tools like the bulk copy program (BCP). In this unit, we'll focus on SQL-based techniques to ingest data from a data lake.

1. Select the  **Data** panel.
2. Within this panel, select the **workspace** tab.
3. Expand the ***SQL Database***
4. On the database created with your **sqlxxxxxxx** suffix mouse-over the right-hand side of the panel until the ellipses appears, then left-click.
5. Select ***New SQL Script***.
6. Select ***Empty Script***.

    ![Select empty sql script](./images/select-empty-sql-script.png)

You now have a blank SQL page, which is connected to the instance for the following exercises.
## Loading data into staging tables

If you use external tables for staging, there's no need to load the data into them because they already reference the data files in the data lake. However, if you use "regular" relational tables, you can use the COPY statement to load data from the data lake, as shown in the following example:

>**NOTE**: Change the ***datalakexxxxxx*** with the name of your datalake name created during the beginning of the lab

```sql
COPY INTO dbo.StageProduct
    (ProductID, ProductAlternateKey, ProductName, ProductCategory, Color, Size, ListPrice, Discontinued)
FROM 'https://datalakexxxxxx.blob.core.windows.net/files/data/StageProduct.csv'
WITH
(
    FILE_TYPE = 'CSV',
    MAXERRORS = 0,
    IDENTITY_INSERT = 'OFF',
    FIRSTROW = 2 --Defines where the first data row starts
);
```

Let's also bring in another table, which will be used for later using the same method.
>**NOTE**: Don't forget to change the ***datalakexxxxxx*** with the name of your datalake name in both the ```FROM``` and the ```ERRORFILE``` elements below.

```sql
COPY INTO dbo.StageCustomers
(CustomerKey, GeographyKey, CustomerAlternateKey, Title, FirstName, MiddleName, LastName, NameStyle, BirthDate, 
MaritalStatus, Suffix, Gender, EmailAddress, YearlyIncome, TotalChildren, NumberChildrenAtHome, EnglishEducation, 
SpanishEducation, FrenchEducation, EnglishOccupation, SpanishOccupation, FrenchOccupation, HouseOwnerFlag, 
NumberCarsOwned, AddressLine1, AddressLine2, Phone, DateFirstPurchase, CommuteDistance)
FROM 'https://datalakemrq3yz8.dfs.core.windows.net/files/data/StageCustomers.csv'
WITH
(
 FILE_TYPE = 'CSV'
 ,MAXERRORS = 0
 ,FIRSTROW = 2 --Defines where the first data row starts
 ,ERRORFILE = 'https://datalakemrq3yz8.dfs.core.windows.net/files/'
)
--END
GO
```

## Loading staged data into dimension tables

Once you've staged and verified the data you can use it to look for changes between the existing and new data (Deltas), perform lookups to detect changes in dimensions, or load it into the dimension tables using SQL.

## Using a CREATE TABLE AS (CTAS) statement

> ***NOTE*** For more Information, see [CREATE TABLE AS SELECT (CTAS)](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-develop-ctas) in the Azure Synapse Analytics documentation.

The CREATE TABLE AS SELECT (CTAS) expression has various uses, which include:
    1. redistributing the hash key of a table to align with other tables for better query performance.
    2. assigning a surrogate key to a staging table based upon existing values after performing a delta analysis.
    3. creating aggregate tables quickly for report purposes.

The statement allows for creating a new table with the results of a SELECT statement.

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

For example, if you have an orders table distributed by order_id, and a transactions table also distributed by order_id. When you join your orders table to your transactions table on order_id, this query becomes a pass-through query. Data movement operations are then eliminated. Fewer steps mean a faster query. Less data movement also makes for faster queries.

The CTAS operation will allow us to use the round-robin table type for loading in those cases and then create a distributed table once the data is understood within the warehouse.

>**NOTE**: Change the ***datalakexxxxxx*** with the name of your datalake name created during the beginning of the lab. If you execute this script prior to changing it you will have to drop and recreate the EXTERNAL DATA SOURCE.
## Using a CREATE EXTERNAL TABLE AS SELECT (CETAS)

```sql
-- Create the external datasource (changing the suffix to match yours
IF NOT EXISTS (SELECT * FROM sys.external_data_sources WHERE name = 'MyDataSource') 
 CREATE EXTERNAL DATA SOURCE [MyDataSource] 
 WITH (
  LOCATION = 'abfss://files@datalakexxxxxxx.dfs.core.windows.net', 
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
    LOCATION='/edata/customer.tbl',  
 DATA_SOURCE = MyDataSource, --The Data Source created above
 FILE_FORMAT = MyParquet --The File Format created above
) AS SELECT * FROM dimCustomer;  

select top 100 * from hdfsCustomer
```

## Updating Dimension tables

There are multiple kinds of slowly changing dimension, of which three are commonly implemented:

* Type 0: Dimension data can't be changed. Any attempted changes fail.
* Type 1: A change made to an existing dimension row applies to all previously loaded facts related to the dimension.
* Type 2: A change to a dimension results in a new dimension row. Existing rows for previous versions of the dimension are retained for historical fact analysis and the new row is applied to future fact table entries.
  
Let's take a look at an example of a Type 1 change. Suppose a store changes its name from "High Street Store" to "Town Center Store". In this case, the change should be reflected for all new sales and also all existing historical sales - so any queries that aggregate sales by store name should include all previous sales in the total for the store, regardless of the name change. To handle this change, the load process must identify the existence of any current rows for the affected store in the dimension table based on the alternate key, and update them to change the store name.

Now let's consider an example of a Type 2 change. Suppose a customer changes their address because they move to a new city. In this case, you would want all existing historical sales to still be counted under the city where the customer lived when the sale was made, and all future sales after they moved to be counted under their new city. To handle this change, the load process must create a new row for the customer with a new surrogate key (but the same alternate key) to reflect the new address. Optionally, the table could include a Boolean column to indicate which record for this alternate key is the currently active record, or a DateTime column to indicate the point in time from which the new record applies (otherwise you can rely on an incrementing surrogate key and use the MAX function to find the most recently inserted row for a given alternate key.).

Logic to implement Type 1 and Type 2 updates can be complex, and there are various techniques you can use. For example, you could use a combination of UPDATE and INSERT statements as shown in the following code example:

```sql
-- Insert new customers noting the schemas of the tables are identical
SET IDENTITY_INSERT dbo.DimCustomer ON

INSERT INTO dbo.DimCustomer ([CustomerKey],[GeographyKey],[CustomerAlternateKey],[Title],[FirstName],[MiddleName],[LastName],[NameStyle],[BirthDate],[MaritalStatus],
[Suffix],[Gender],[EmailAddress],[YearlyIncome],[TotalChildren],[NumberChildrenAtHome],[EnglishEducation],[SpanishEducation],[FrenchEducation],
[EnglishOccupation],[SpanishOccupation],[FrenchOccupation],[HouseOwnerFlag],[NumberCarsOwned],[AddressLine1],[AddressLine2],[Phone],
[DateFirstPurchase],[CommuteDistance])
SELECT *
FROM dbo.StageCustomers AS stg
WHERE NOT EXISTS
    (SELECT * FROM dbo.DimCustomer AS dim
     WHERE dim.CustomerKey = stg.CustomerKey);

SET IDENTITY_INSERT dbo.DimCustomer OFF

--Look for type 1 updates in our staging file
SELECT dim.LastName, stg.LastName, dim.EmailAddress, stg.EmailAddress, dim.Phone, stg.Phone
FROM DimCustomer dim inner join StageCustomers stg
ON dim.CustomerKey = stg.CustomerKey
WHERE dim.LastName <> stg.LastName OR dim.EmailAddress <> stg.EmailAddress OR dim.Phone <> stg.Phone

-- Type 1 updates (name, email, phone)
UPDATE dbo.DimCustomer
SET dim.LastName = stg.LastName,
    dim.EmailAddress = stg.EmailAddress,
    dim.Phone = stg.Phone
FROM DimCustomer dim inner join StageCustomers stg
ON dim.CustomerKey = stg.CustomerKey
WHERE dim.LastName <> stg.LastName OR dim.EmailAddress <> stg.EmailAddress OR dim.Phone <> stg.Phone

-- Type 2 updates (geographic address changes triggers new entry)
INSERT INTO dbo.DimCustomer
SELECT stg.GeographyKey,stg.CustomerAlternateKey,stg.Title,stg.FirstName,stg.MiddleName,stg.LastName,stg.NameStyle,stg.BirthDate,stg.MaritalStatus,
stg.Suffix,stg.Gender,stg.EmailAddress,stg.YearlyIncome,stg.TotalChildren,stg.NumberChildrenAtHome,stg.EnglishEducation,stg.SpanishEducation,stg.FrenchEducation,
stg.EnglishOccupation,stg.SpanishOccupation,stg.FrenchOccupation,stg.HouseOwnerFlag,stg.NumberCarsOwned,stg.AddressLine1,stg.AddressLine2,stg.Phone,
stg.DateFirstPurchase,stg.CommuteDistance
FROM dbo.StageCustomers AS stg
JOIN dbo.DimCustomer AS dim
ON stg.CustomerKey = dim.CustomerKey
WHERE stg.AddressLine1 <> dim.AddressLine1 OR stg.AddressLine2 <> dim.AddressLine2;
```

>**NOTE** In the previous example, it is assumed that an incrementing surrogate key based on an ```IDENTITY``` column identifies each row, and that the highest value surrogate key for a given alternate key indicates the most recent or "current" instance of the dimension entity associated with that alternate key. In practice, many data warehouse designers include a Boolean column to indicate the current active instance of a changing dimension or use DateTime fields to indicate the active time periods for each version of the dimension instance. With these approaches, the logic for a type 2 change must include an ```INSERT``` of the new dimension row and an ```UPDATE```to mark the current row as inactive

As an alternative to using multiple ```INSERT``` and ```UPDATE``` statement, you can use a single ```MERGE``` statement to perform an "UPSERT" operation to insert new records and update existing ones, as shown in the following example, which loads new product records and applies type 1 updates to existing products:

```sql
MERGE dbo.DimProduct AS tgt
    USING (SELECT * FROM dbo.StageProducts) AS src
    ON src.ProductID = tgt.ProductBusinessKey
WHEN MATCHED THEN
    UPDATE SET
        tgt.ProductName = src.ProductName,
        tgt.ProductCategory = src.ProductCategory
        tgt.Color = src.Color,
        tgt.Size = src.Size,
        tgt.ListPrice = src.ListPrice,
        tgt.Discontinued = src.Discontinued
WHEN NOT MATCHED THEN
    INSERT VALUES
        (src.ProductID,
         src.ProductName,
         src.ProductCategory,
         src.Color,
         src.Size,
         src.ListPrice,
         src.Discontinued);
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

>**NOTE** For more information, see the [Indexes on dedicated SQL pool tables in Azure Synapse Analytics](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-index) and [Table statistics for dedicated SQL pool in Azure Synapse Analytics articles](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-statistics) in the Azure Synapse Analytics documentation.

## Delete Azure resources

If you've finished exploring Azure Synapse Analytics, you should delete the resources you've created to avoid unnecessary Azure costs.

1. Close the Synapse Studio browser tab and return to the Azure portal.
2. On the Azure portal, on the **Home** page, select **Resource groups**.
3. Select the **dp000-*xxxxxxx*** resource group for your Synapse Analytics workspace (not the managed resource group), and verify that it contains the Synapse workspace, storage account, and Spark pool for your workspace.
4. At the top of the **Overview** page for your resource group, select **Delete resource group**.
5. Enter the **dp000-*xxxxxxx*** resource group name to confirm you want to delete it, and select **Delete**.

    After a few minutes, your Azure Synapse workspace resource group and the managed workspace resource group associated with it will be deleted.
