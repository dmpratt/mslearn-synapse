---
lab:
    title: 'Transform data using a serverless SQL pool'
    module: 'Use Azure Synapse serverless SQL pool to query files in a data lake'
---

# Transform files using a serverless SQL pool

Data *engineers* often use Spark notebooks as one of their preferred tools to perform *extract, load, and transformation (ELT)* activities.

In this lab, you'll use a serverless SQL pool in Azure Synapse Analytics to transform data in files.

This lab will take approximately **30** minutes to complete.

## Before you start

You'll need an [Azure subscription](https://azure.microsoft.com/free) in which you have administrative-level access.

## Provision an Azure Synapse Analytics workspace

You'll need an Azure Synapse Analytics workspace with access to data lake storage. You can use the built-in serverless SQL pool to query files in the data lake.

In this exercise, you'll use a combination of a PowerShell script and an ARM template to provision an Azure Synapse Analytics workspace.

1. Sign into the [Azure portal](https://portal.azure.com) at `https://portal.azure.com`.
2. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new Cloud Shell in the Azure portal, selecting a ***PowerShell*** environment and creating storage if prompted. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal, as shown here:

    ![Azure portal with a cloud shell pane](./images/cloud-shell.png)

    > **Note**: If you have previously created a cloud shell that uses a *Bash* environment, use the the drop-down menu at the top left of the cloud shell pane to change it to ***PowerShell***.

3. Note that you can resize the cloud shell by dragging the separator bar at the top of the pane, or by using the **&#8212;**, **&#9723;**, and **X** icons at the top right of the pane to minimize, maximize, and close the pane. For more information about using the Azure Cloud Shell, see the [Azure Cloud Shell documentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. In the PowerShell pane, enter the following commands to clone this repo:

    ```
    rm -r dp-000 -f
    git clone https://github.com/MicrosoftLearning/mslearn-synapse dp-000
    ```

5. After the repo has been cloned, enter the following commands to change to the folder for this lab and run the **setup.ps1** script it contains:

    ```
    cd dp-000/Allfiles/Labs/14
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
4. On the Manage page, select the Apache Spark pools tab and note that a Spark pool with a name similar to **sparkxxxxxxx** has been provisioned in the workspace.
5. On the Data page, view the Linked tab and verify that your workspace includes a link to your Azure Data Lake Storage Gen2 storage account, which should have a name similar to **synapsexxxxxxx (Primary - datalakexxxxxxx)**.
6. Expand your storage account and verify that it contains a file system container named **files (primary)**.
7. Select the files container, and note that it contains folders named sales and synapse. The synapse folder is used by Azure Synapse, and the sales folder contains the data files you are going to query.
Open the sales folder and the orders folder it contains, and observe that the orders folder contains .csv files for three years of sales data.
***Right-click*** any of the files and select Preview to see the data it contains. Note that the files contain a header row, so you can select the option to display column headers.
8. Select the Develop Panel and expand the Notebooks section.
9. Select your spark server and run the first code cell which will take several minutes to execute.
10. complete the rest of the code steps in the notebook and then return to this page.


## View and Work with the created parquet files 
In the Last step of the notebook we queried the parquet files Partitioned by FiscalYear and Fiscal Month. In total, there were 30 parquet files created which represented 30 months of data from 2020 thru June of 2022. Let's explore these files a little more.
1. On the left side of Synapse Studio, if not alreaddy done so, use the ›› icon to expand the menu - this reveals the different pages within Synapse Studio that you’ll use to manage resources and perform data analytics tasks.
2. On the Data page, view the Linked tab and verify that your workspace includes a link to your Azure Data Lake Storage Gen2 storage account, which should have a name similar to **synapsexxxxxxx (Primary - datalakexxxxxxx)**.
3. Expand your storage account and open the file system container named **files (primary)**.
4. double-click on the **data** folder then double-click on the ***OrdersTransform.parquet*** folder
5. You will see folders named ***FiscalYear=2020, FiscalYear=2021, FiscalYear=2022*** double-click on any of these and you will see folders named ***FiscalMonth=1*** and up through 12 except in the 2022 Year which goes up to 6.  
6. Double-click on any of these ***FiscalMonth*** folders and you will see the actual parquet file.
7. Right Click on the parque file in the folder and select ***New notebook > Load to DataFrame
    > **WARNING**: Don't execute this code here - you will likely not have enough spark cores to complete and it will result in an error
8. Click in the code window of the new DataFrame and press ctrl + a to highlight all of the code, then use ctrl + c to copy the code
9. Across the top of you notebooks you will see the tabs, **Spark Tranform | files | Notebook 1**, click on **Spark Transform and then scroll to the bottom if it's not already there.
10. Click on the **+ Code** button by hovering on the bottom left of the last execution window and then press ctrl + v to paste the generated code into this new code window. 
11. You can execute with hitting the ***shift key + enter key*** or pressing the ***Run cell*** button on the left side of the code panel.
    > **NOTE**: Your session might have expired, if so, just run the code again and wait several minutes (up to 5 mins) for the Apache Spark session to resume.
12. As this is a parquet format, you can read the data for the entire year, simply by removing the path behind Fiscal Year as shown below:
    ```Python

    df = spark.read.load('abfss://files@datalakexxxxx.dfs.core.windows.net/data/OrdersTransform.parquet/FiscalYear=2020/FiscalMonth=1/part-00000-aa61d52e-d7d0-47c5-aad0-ba2038f296e6.c000.snappy.parquet', format='parquet')

    #Change to 

    df = spark.read.load('abfss://files@datalakexxxx.dfs.core.windows.net/data/OrdersTransform.parquet/FiscalYear=2020', format='parquet')
    
    ```
This completes the lab, feel free to explore further with this environment and then be sure to follow the steps below before moving forward.
## Delete Azure resources

If you've finished exploring Azure Synapse Analytics, you should delete the resources you've created to avoid unnecessary Azure costs.

1. Close the Synapse Studio browser tab and return to the Azure portal.
2. On the Azure portal, on the **Home** page, select **Resource groups**.
3. Select the **dp000-*xxxxxxx*** resource group for your Synapse Analytics workspace (not the managed resource group), and verify that it contains the Synapse workspace, storage account, and Spark pool for your workspace.
4. At the top of the **Overview** page for your resource group, select **Delete resource group**.
5. Enter the **dp000-*xxxxxxx*** resource group name to confirm you want to delete it, and select **Delete**.

    After a few minutes, your Azure Synapse workspace resource group and the managed workspace resource group associated with it will be deleted.
