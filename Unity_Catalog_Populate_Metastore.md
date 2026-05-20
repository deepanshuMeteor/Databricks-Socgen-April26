---
lab:
  title: Populate and navigate the Metastore in Unity Catalog
  description: You'll gain hands-on experience creating and managing Unity Catalog data objects by building catalogs, schemas, tables, and views using SQL commands, then navigating the three-level namespace hierarchy through both SQL queries (SHOW CATALOGS, DESCRIBE EXTENDED) and Catalog Explorer. You'll learn how Unity Catalog tracks object dependencies and lineage, and understand how to use different approaches—programmatic SQL and visual interface—to inspect and manage metastore metadata.
  duration: 30 minutes
  level: 300
  islab: true
---

# Populate and navigate the Metastore in Unity Catalog

Unity Catalog provides a centralized governance solution for data assets in AWS Databricks. The metastore serves as the top-level container in Unity Catalog, organizing data objects using a three-level namespace that provides boundaries for data isolation and access control.

In this exercise, you'll create catalogs using Catalog Explorer, populate schemas and tables using SQL commands, then navigate the hierarchy using both SQL queries and the Catalog Explorer interface.

This exercise should take approximately **30** minutes to complete.



## Open your AWS Databricks workspace



1. Open your  Databricks workspace (named **pikapika** )



## Explore the default catalog

1. In the left sidebar menu, click **Catalog** to open Catalog Explorer.

2. Notice that some catalogs are already created for you:
   - A **system** catalog that contains built-in metadata
   - A default catalog with the same name as your workspace (e.g., **pikapika,**)

3. Click on the default catalog (the one with the same name as your workspace) to explore its structure.

4. Notice that two schemas are already created in the default catalog:
   - **default** - A default schema for organizing data objects
   - **information_schema** - A system schema containing metadata

5. Click the **Details** button in the catalog details panel and observe the **Storage location** and **Type** fields. The type **MANAGED_CATALOG** indicates that AWS Databricks automatically manages the storage and lifecycle of data assets within this catalog.

## Create a new catalog

Now that you've explored the default catalog, you'll create your own catalog for organizing data.

1. Select **Catalog** from the left menu.

2. Select **Catalogs** below **Quick access**.
3. Select **Create catalog**.
4. In the **Create a new catalog** dialog:
   - Enter a **Catalog name** (for example, `my_catalog_(yourname)`)
   - Select **Standard** as the catalog **Type**
   - For **Storage location**, select your default catalog name (e.g., **pikapika**) to use the same storage location it uses.
   - Click **Create**

3. In the **Catalog created** window that appears, click **View catalog**.

4. Click on your newly created catalog to view its structure. Notice it contains default **default** and **information_schema** schemas, similar to the workspace catalog you explored earlier.

## Create a Notebook

You'll use a notebook to run SQL commands that create and explore Unity Catalog objects in your new catalog.

1. In the sidebar, use the **(+) New** link to create a **Notebook**.
   
2. Change the default notebook name (**Untitled Notebook *[date]***) to `Populate and navigate the metastore` and in the **Connect** drop-down list, select the **Serverless cluster** if it is not already selected. Note that **Serverless** is enabled by default.

3. In the first cell of your notebook, enter and run the following code to set your new catalog as the default and verify:

    ```
    %sql
    USE CATALOG <your_catalog_name>;
    SELECT current_catalog();
    ```

## Create and manage schemas

1. Add a new cell and run the following code to create a schema named **sales** and set it as default:

    ```
    %sql
    CREATE SCHEMA IF NOT EXISTS sales
    COMMENT 'Schema for sales data';

    USE SCHEMA sales;
    SELECT current_schema();
    ```

2. In **Catalog Explorer**, navigate to your catalog and expand it to see the **sales** schema you just created alongside the **default** and **information_schema** schemas.

3. Return to your notebook by selecting **Workspace** from the left menu, then navigating to your notebook.

## Create and manage tables

1. Add a new cell and run the following code to create a managed table for customer data:

    ```
    %sql
    CREATE OR REPLACE TABLE customers (
      customer_id INT,
      customer_name STRING,
      email STRING,
      city STRING,
      country STRING
    )
    COMMENT 'Customer information table';
    ```

2. Add a new cell and run the following code to insert sample data and verify it was inserted:

    ```
    %sql
    INSERT INTO customers VALUES
      (1, 'Aaron Gonzales', 'aaron@contoso.com', 'Seattle', 'USA'),
      (2, 'Anne Patel', 'anne@contoso.com', 'London', 'UK'),
      (3, 'Casey Jensen', 'casey@contoso.com', 'Toronto', 'Canada'),
      (4, 'Elizabeth Moore', 'elizabeth@contoso.com', 'Sydney', 'Australia'),
      (5, 'Liam Davis', 'liam@contoso.com', 'Berlin', 'Germany');

    SELECT * FROM customers;
    ```
3. Switch to **Catalog Explorer** and navigate to your catalog > **sales** schema > **customers** table. Click on the table to explore:
   - **Schema** tab - View the column definitions and data types
   - **Sample Data** tab - See a preview of the data you inserted
   - **Details** tab - View table metadata including storage location and table type (MANAGED)

4. Return to your notebook by selecting **Workspace** from the left menu, then navigating to your notebook.

## Create and manage views

1. Add a new cell and run the following code to create a view that filters customers:

    ```
    %sql
    CREATE OR REPLACE VIEW usa_customers AS
    SELECT customer_id, customer_name, email, city
    FROM customers
    WHERE country = 'USA';

    SELECT * FROM usa_customers;
    ```

2. Switch to **Catalog Explorer** and navigate to your catalog > **sales** schema. Notice that both the **customers** table and **usa_customers** view are now listed.

3. Click on the **usa_customers** view and select the **Lineage** tab, then select **See lineage graph**. Observe how the lineage graph shows the dependency relationship between the view and its source table. Unity Catalog tracks these object dependencies, which helps you understand data flow and assess the impact of changes to underlying tables.

## Explore the metastore with SQL commands

Now that you've created objects using SQL and verified them in Catalog Explorer, let's use SQL commands to inspect the metastore structure programmatically.

1. Return to your notebook by selecting **Workspace** from the left menu, then navigating to your notebook.

2. Add a new cell and run the following code to list all catalogs you have access to:

    ```
    %sql
    SHOW CATALOGS;
    ```

   This lists all catalogs you have access to, including the **system** catalog, your workspace catalog, and your custom catalog.

3. Add a new cell and run the following code to list all schemas in your current catalog:

    ```
    %sql
    SHOW SCHEMAS;
    ```

4. Add a new cell and run the following code to list all tables and views in your current schema:

    ```
    %sql
    SHOW TABLES;
    ```

5. Add a new cell and run the following code to use DESCRIBE to get detailed table metadata:

    ```
    %sql
    DESCRIBE EXTENDED customers;
    ```

   The DESCRIBE EXTENDED command provides comprehensive information about the table including column definitions, table properties, storage location, and more.

6. Switch back to **Catalog Explorer** one final time. Compare what you see in the visual interface with the SQL output. Notice how both approaches - SQL commands and Catalog Explorer - provide different views of the same metadata, giving you flexibility in how you navigate and manage your Unity Catalog metastore.

