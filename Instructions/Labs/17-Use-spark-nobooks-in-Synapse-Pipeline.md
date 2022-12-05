---
lab:
    title: 'Use spark notebooks in Synapse Pipeline'
    module: 'Transfer and transform data with Azure Synapse Analytics Pipelines'
---
## Use spark notebooks in Synapse Pipeline

## Introduction

In this lab, we're going to load data into a dedicated SQL Pool using the built-in Synapse Analytics Pipeline located within Azure Synapse Analytics Explorer. This lab will start with a basic copy action and then built up from there

This lab will take approximately **45** minutes to complete.

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
    cd dp-000/Allfiles/Labs/16
    ./setup.ps1
    ```
    
6. If prompted, choose which subscription you want to use (this will only happen if you have access to multiple Azure subscriptions).
7. When prompted, enter a suitable password to be set for your Azure Synapse SQL pool.

    > **Note**: Be sure to remember this password!

8. Wait for the script to complete - this typically takes around 10 minutes, but in some cases may take longer. While you're waiting, review the [Apache Spark Pool Configurations](https://learn.microsoft.com/en-us/azure/synapse-analytics/spark/apache-spark-pool-configurations) article in the Azure Synapse Analytics documentation.

## View and Navigate Synapse Workspace

1. After the script has completed, in the Azure portal, go to the dp000-xxxxxxx resource group that it created, and select your Synapse workspace.
2. In the Overview page for your Synapse Workspace, in the Open Synapse Studio card, select Open to open Synapse Studio in a new browser tab; signing in if prompted.
3. On the left side of Synapse Studio, use the ›› icon to expand the menu - this reveals the different pages within Synapse Studio that you’ll use to manage resources and perform data analytics tasks.
4. On the Data page, view the Linked tab and verify that your workspace includes a link to your Azure Data Lake Storage Gen2 storage account, which should have a name similar to **synapsexxxxxxx (Primary - datalakexxxxxxx)**.
5. Expand your storage account and verify that it contains a file system container named **files (primary)**.
6. Select the files container, and note that it contains folders named data and synapse. The synapse folder is used by Azure Synapse, and the data folder contains the data files you're going to query.
Open the sales folder and the orders folder it contains, and observe the files contained within it.
***Right-click*** any of the files and select Preview to see the data it contains. Note if the files contain a header row, so you can determine whether to select the option to display column headers.
## Execute a notebook in an Azure Synapse Pipeline

Earlier in a lab we created a Synapse Notebook and created some tables. We're going to return back to that notebook and add some code that allows us to call and run a step within the notebook as part of our pipeline using a parameter.
## Create a parameter cell

1. On the **Develop** tab, select the notebook named **Spark Transform**
2. in the Right panel of the **Spark Transform** notebook, click on the drop-down box to the right of the **Attach to**
3. Select **sparkxxxxxxx** replacing the "x" elements with the suffix assigned in your setup script
4. Below the section in the notebook named **Load Source Data** and below the code block that populates the ***order_details*** python dataframe, click on the **+ Code**.
5. Type or paste the following commands into the new code window.

```python
# We're going to use this for integration into a Synapse Pipleline
import uuid

#generates a random GUID
runId = uuid.uuid4()
print("Your parquet file to be created: " + str(runId))
```

4. Mouse-over the code and select the **...** ellipse on the right-side of the code block then select **Toggle parameter cell**. You will notice the text ***parameters*** now appear in the bottom-right or top-right corner of the code block.

    ![Changing uuid to a parameter](images/select-as-parameter.png)

5. Below this code block select the **+ Code** to create another code block.
6. Type or paste the following commands into the new window

```python
%%pyspark

order_details.write.parquet('abfss://files@datalakexxxxxxx.dfs.core.windows.net/data/order_details/' + str(runId) + '.parquet')
```

7. Replace the text in the new code **datalakexxxxxxx** with the name of the data lake within your **Resource Group** during the lab build.
8. On the top right of the notebook pane, select the pipeline icon.
9. select the **New pipeline** option. 

    ![New pipeline](images/new-notebook-pipeline.png)

