
# Upgrade Tables to Unity Catalog

Unity Catalog provides a centralized governance solution for data assets in AWS Databricks. Unity Catalog delivers fine-grained access controls, automated data lineage tracking, and cross-workspace data sharing capabilities that build on the basic table management that any Catalog provides.

In this exercise, you'll learn how to upgrade existing tables from any Catalog to Unity Catalog. You'll use SQL commands and the AWS Databricks user interface to analyze existing data structures, apply migration techniques, evaluate transformation options, and upgrade metadata without moving data.

This exercise should take approximately **30** minutes to complete.



## Review what's in catalog explorer
1. Ensure that you are in workspace named - **pikapika**
2. Select **Catalog** on the left of your workspace.
3. Note that there's a catalog that was automatically created and has the same name as your workspace.
4. Expand that catalog and note the default schema. We'll migrate tables to that default schema later in the exercise.
5. Note the catalog that you have created. 
6. Open the **samples** catalog, and note the **bakehouse** schema. Expand it and note the table named sales_customers.
7. In this exercise, we'll use sample data from the samples.bakehouse.sales_customers table to create and populate a table in the pikapika_ingestion_data Catalog. Then we'll explore options to upgrade that table to Unity Catalog.

## Create a Notebook
You'll use a notebook to run SQL commands that demonstrate various table upgrade techniques.

1. In the sidebar, use the **(+) New** link to create a **Notebook**.
   
2. Change the default notebook name (**Untitled Notebook *[date]***) to `Upgrade tables to Unity Catalog` and in the **Connect** drop-down list, select **Serverless.**

## Create objects
Load sample data into a sample catalog(the one you created in previous exercise) so you can migrate it to Unity Catalog.

1. Add a new cell and run the following code to create a schema and a populated table in your catalog, then confirm the rows are in the new table:

    ```sql
    CREATE SCHEMA IF NOT EXISTS <yourcatalog_name>.bakehouse;

    CREATE TABLE <yourcatalog_name>.bakehouse.sales_customers AS
    SELECT * FROM samples.bakehouse.sales_customers;

    SELECT * FROM <yourcatalog_name>.bakehouse.sales_customers;
    ```

2.  In a new cell, run the following to set your default catalog and schema to the Unity Catalog that has the same name as your workspace.
    
    **Note**: The name of your workspace is on the top right of your AWS Databricks workspace. Replace `<workspace_catalog>` in the USE CATALOG statement below with the name of your workspace. If your workspace name has a `-` in it, replace that with a `_`.

    ```sql
    USE CATALOG <workspace_catalog>;
    USE SCHEMA default;
    SELECT current_catalog(), current_schema()
    ```  

## Overview of upgrade methods

There are a few different ways to upgrade a table, but the method you choose will be driven primarily by how you want to treat the table data. If you want to leave the table data in place, then the resulting upgraded table will be an external table. If you want to move the table data into your Unity Catalog metastore, then the resulting table will be a managed table.

### Move table data into the Unity Catalog Metastore

In this approach, table data will be copied from wherever it resides into the managed data storage area for the destination schema. The result will be a managed Delta table in your Unity Catalog metastore.

#### Clone a Table

Cloning a table is optimal when the source table is Delta. It's simple to use, will copy metadata, and gives you the option of copying data (deep clone) or leaving it in place (shallow clone).

1. Add a new cell and run the following code to check the format of the source table:

    ```sql
    DESCRIBE EXTENDED <yourcatalog_name>.bakehouse.sales_customers;
    ```

   Notice that the *Provider* row shows the source is a Delta table, and the *Location* row shows that the table is stored in DBFS.

2. Add a new cell and run the following code to perform a deep clone operation. This will create a table in the default catalog in Unity Catalog. Add a new cell and run the following code to clone the catalog metastore into a new table in Unity Catalog. The new table will be created in the default schema of catalog that has the same name as your workspace.  

    ```sql
    CREATE OR REPLACE TABLE sales_customers_clone
      DEEP CLONE <yourcatalog_name>.bakehouse.sales_customers;
    ```

3. Verify the cloned table by viewing your catalog in Catalog Explorer:
   - Select the **Catalog** icon on the left side of your workspace
   - Expand your catalog with the same name as your workspace
   - Expand the **default** schema
   - Notice that the **sales_customer** table has been cloned as **sales_customer_clone**

#### Create Table As Select (CTAS)

Using CTAS is a universally applicable technique that creates a new table based on the output of a `SELECT` statement. This will always copy the data, but no metadata will be copied.

1. Return to the notebook. Add a new cell and run the following code to copy the table using CTAS:

    ```sql
    CREATE TABLE sales_customers_ctas AS 
    SELECT * 
    FROM <yourcatalog_name>.bakehouse.sales_customers;
    ```

2. Add a new cell and run the following code to verify the table was created:

    ```sql
    SHOW TABLES IN default;
    ```

#### Apply transformations during the upgrade

CTAS offers the ability to transform the data while copying it. When migrating tables to Unity Catalog, it's a great time to consider whether your table structures still address your organization's business requirements.

1. Add a new cell and run the following code to create a transformed version of the table:

    ```sql
    CREATE TABLE sales_customers_transformed AS 
    SELECT
      customerID as ID,
      first_name as first_name,
      last_name as last_name
    FROM <yourcatalog_name>.sales_customers;
    ```

2. Verify the new table by viewing your catalog in Catalog Explorer:
   - Select the **Catalog** icon on the left side of your workspace
   - Expand your catalog with the same name as your workspace
   - Expand the **default** schema
   - Expand **Tables**
   - Select the **sales_customer_transformed** table and note that the ID column name has been changed from customerID. If you don't see the table, select the refresh button at the top of Catalog Explorer.

## Upgrade external tables (Example)

**Note**: You may not have access to external tables so this is an example of what you can do in your environment.

When upgrading external tables, some use cases may call for leaving the data in place, such as when data location is dictated by regulatory requirements, you cannot change the data format to Delta, or you want to avoid the time and cost of moving large datasets.

