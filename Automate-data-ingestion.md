
# Automate data ingestion and processing using AWS Databricks

Databricks Jobs is a powerful service that allows for the automation of data ingestion and processing workflows. It enables the orchestration of complex data pipelines, which can include tasks like ingesting raw data from various sources, transforming this data using Delta Live Tables, and persisting it to Delta Lake for further analysis. With AWSe Databricks, users can schedule and run their data processing tasks automatically, ensuring that data is always up-to-date and available for decision-making processes.

This lab will take approximately **30** minutes to complete.



## Create a notebook and get source data

1. In the sidebar, use the **(+) New** link to create a **Notebook** and change the default notebook name (**Untitled Notebook *[date]***) to **Data Processing**. Then, in the **Connect** drop-down list, select your cluster if it is not already selected. If the cluster is not running, it may take a minute or so to start.

2. In the first cell of the notebook, enter the following code, which uses *shell* commands to download data files from GitHub into the file system used by your cluster.

     ```python
    %sh
    %sh
    rm -rf /Workspace/Users/<your-email-address>/Filestore
    mkdir -p /Workspace/Users/<your-email-address>/Filestore
    wget -O /Workspace/Users/<your-email-address>/Filestore/sample_sales_data.csv https://raw.githubusercontent.com/Kiran-255666/datasets/refs/heads/main/sample_sales_data.csv
     ```

3. Use the **&#9656; Run Cell** menu option at the left of the cell to run it. Then wait for the Spark job run by the code to complete.

## Automate data processing with AWS Databricks Jobs

1. Replace the code in the first of the notebook, with the following code. Then run it to load the data into a dataframe:

     ```python
    # Load the sample dataset into a DataFrame
    df = spark.read.csv('/Workspace/Users/<your-email-address>/FileStore/*.csv', header=True, inferSchema=True)
    df.show()
     ```

1. Move the mouse under the existing code cell, and use the **+ Code** icon that appears to add a new code cell. Then in the new cell, enter and run the following code to aggregate sales data by product category:

     ```python
    from pyspark.sql.functions import col, sum

    # Aggregate sales data by product category
    sales_by_category = df.groupBy('product_category').agg(sum('transaction_amount').alias('total_sales'))
    sales_by_category.show()
     ```

1. In the sidebar, use the **(+) New** link to create a **Job**.

1. Change the default job name (**New job *[date]***) to `Automated job`.

1. Configure the unnamed task in the job with the following settings:
    - **Task name**: `Run notebook`
    - **Type**: Notebook
    - **Source**: Workspace
    - **Path**: *Select your* Data Processing *notebook*
    - **Cluster**: *Select your cluster(our shared cluster)*

1. Select **Create task**.

1. Select **Run now**

    **Tip**: In the right-side panel, under **Schedule**, you can select **Add trigger** and set up a schedule for running the job (e.g., daily, weekly). However, for this exercise, we will execute it manually.

1. Select the **Runs** tab in the Job panel and monitor the job run.

1. After the job run is successful, you can select it in the **Runs** list and verify its output.

    You have successfully set up and automated data ingestion and processing using AWS Databricks Jobs. You can now scale this solution to handle more complex data pipelines and integrate with other AWS services for a robust data processing architecture.

