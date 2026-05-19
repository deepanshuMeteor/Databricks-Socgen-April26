---
lab:
  title: Deploy workloads with Databricks Lakeflow jobs
  description: You'll gain hands-on experience automating data workflows by creating Lakeflow jobs that orchestrate notebook tasks for ETL operations, including data ingestion, cleaning, and aggregation. You'll learn how to configure job tasks to run notebooks on serverless compute, monitor job execution through the Jobs & Pipelines interface, and understand how to schedule periodic job runs using triggers for production workloads.
  duration: 40 minutes
  level: 400
  islab: true

---

Keep the Databricks Workspace open

## Create a notebook

1. In the sidebar, use the **(+) New** link to create a **Notebook**.

1. Change the default notebook name (**Untitled Notebook *[date]***) to `ETL task` and in the **Connect** drop-down list, select **Serverless** compute if it is not already selected. If the compute is not running, it may take a minute or so to start.

## Ingest data

1. In the first cell of the notebook, enter the following code to create a volume for storing some lab files.

    ```sql
    %sql
    CREATE VOLUME IF NOT EXISTS spark_lab
    ```

1. Add a new code cell and use it to run the following code, which uses *Python* to download data files from GitHub into your volume.

    ```python
    import requests

    # Define the current catalog
    catalog_name = spark.sql("SELECT current_catalog()").collect()[0][0]

    # Define the base path using the current catalog
    volume_base = f"/Volumes/{catalog_name}/default/spark_lab"

    # List of files to download
    files = ["2019.csv", "2020.csv", "2021.csv"]

    # Download each file
    for file in files:
        url = f"https://raw.githubusercontent.com/MicrosoftLearning/mslearn-databricks/main/data/{file}"
        response = requests.get(url)
        response.raise_for_status()

        # Write to Unity Catalog volume
        with open(f"{volume_base}/{file}", "wb") as f:
            f.write(response.content)
    ```

1. Use the **&#9656; Run Cell** menu option at the left of the cell to run it. Then wait for the Spark job run by the code to complete.
1. Under the output, use the **+ Code** icon to add a new code cell, and use it to run the following code, which defines a schema for the data:

    ```python
   from pyspark.sql.types import *
   from pyspark.sql.functions import *
   orderSchema = StructType([
        StructField("SalesOrderNumber", StringType()),
        StructField("SalesOrderLineNumber", IntegerType()),
        StructField("OrderDate", DateType()),
        StructField("CustomerName", StringType()),
        StructField("Email", StringType()),
        StructField("Item", StringType()),
        StructField("Quantity", IntegerType()),
        StructField("UnitPrice", FloatType()),
        StructField("Tax", FloatType())
   ])
   df = spark.read.load(f'/Volumes/{catalog_name}/default/spark_lab/*.csv', format='csv', schema=orderSchema)
   display(df.limit(100))
    ```

## Create a job task

1. Under the existing code cell, use the **+ Code** icon to add a new code cell. Then in the new cell, enter and run the following code to remove duplicate rows and to replace the `null` entries with the correct values:

     ```python
    from pyspark.sql.functions import col
    df = df.dropDuplicates()
    df = df.withColumn('Tax', col('UnitPrice') * 0.08)
    df = df.withColumn('Tax', col('Tax').cast("float"))
     ```
    > **Note**: After updating the values in the **Tax** column, its data type is set to `float` again. This is due to its data type changing to `double` after the calculation is performed. Since `double` has a higher memory usage than `float`, it is better for performance to type cast the column back to `float`.

2. In a new code cell, run the following code to aggregate and group the order data:

    ```python
   yearlySales = df.select(year("OrderDate").alias("Year")).groupBy("Year").count().orderBy("Year")
   display(yearlySales)
    ```

## Build the Workflow

Databricks manages the task orchestration, cluster management, monitoring, and error reporting for all of your jobs. You can run your jobs immediately, periodically through an easy-to-use scheduling system, whenever new files arrive in an external location, or continuously to ensure an instance of the job is always running.

1. In your workspace, click ![Workflows icon.](./images/WorkflowsIcon.svg) **Jobs & Pipelines** in the sidebar.

2. In the Jobs & Pipelines pane, select **Create**, then **Job**.

3. Change the default job name (**New job *[date]***) to `ETL job`.

4. Configure the job with the following settings:
    - **Task name**: `Run ETL task notebook`
    - **Type**: Notebook
    - **Source**: Workspace
    - **Path**: *Select your* ETL task *notebook*
    - **Cluster**: *Select Serverless*

5. Select **Create task**.

6. Select **Run now**.

7. After the job starts running, you can monitor its execution by selecting **Job Runs** in the left sidebar.

8. After the job run succeeds, you can select it and verify its output.

Additionally, you can run jobs on a triggered basis, for example, running a workflow on a schedule. To schedule a periodic job run, you can open the job task and add a trigger.
