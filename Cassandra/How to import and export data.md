# How to import and export data

## Table of Contents
- [How to import and export data](#how-to-import-and-export-data)
  - [Table of Contents](#table-of-contents)
  - [Architecture Overview](#architecture-overview)
    - [Peer-to-peer architecture](#peer-to-peer-architecture)
    - [Check the replication factor and consistency settings in Cassandra](#check-the-replication-factor-and-consistency-settings-in-cassandra)
    - [For a high-performance data migration into Cassandra](#for-a-high-performance-data-migration-into-cassandra)
    - [Tools for importing and exporting data](#tools-for-importing-and-exporting-data)
    - [To upload data into a Cassandra database after generating stage tables](#to-upload-data-into-a-cassandra-database-after-generating-stage-tables)
  - [CQLSH (Cassandra Query Language Shell):](#cqlsh-cassandra-query-language-shell)
    - [Export data from a table in your Cassandra container](#export-data-from-a-table-in-your-cassandra-container)
    - [Error indicates that the `/data` directory doesn't exist](#error-indicates-that-the-data-directory-doesnt-exist)
    - [Copy the File from the Container to the Host](#copy-the-file-from-the-container-to-the-host)
    - [COPY command in cqlsh to export each table](#copy-command-in-cqlsh-to-export-each-table)
    - [make it executable,](#make-it-executable)
    - [Editor text](#editor-text)
  - [DataStax Bulk Loader (DSBulk)](#datastax-bulk-loader-dsbulk)
    - [How to install](#how-to-install)
    - [the `datastax/dsbulk` image](#the-datastaxdsbulk-image)
    - [Confirm Port Accessibility](#confirm-port-accessibility)
    - [DSBulk in your user folder](#dsbulk-in-your-user-folder)
    - [check if the ports are available](#check-if-the-ports-are-available)
    - [Check Kubernetes ports](#check-kubernetes-ports)
  - [Use Cassandra Bulk Loader (cassandra-loader or sstableloader) \[1\]](#use-cassandra-bulk-loader-cassandra-loader-or-sstableloader-1)
  - [Spring Batch and Spring Data for Apache Cassandra](#spring-batch-and-spring-data-for-apache-cassandra)
    - [Steps to Insert Data into Cassandra using Spring Batch](#steps-to-insert-data-into-cassandra-using-spring-batch)
    - [Key Points:](#key-points)
  - [Kafka and Cassandra Integration: Apache Kafka as an intermediary between the stage tables and Cassandra could be an option.](#kafka-and-cassandra-integration-apache-kafka-as-an-intermediary-between-the-stage-tables-and-cassandra-could-be-an-option)
  - [Decrypt your encrypted data in Cassandra](#decrypt-your-encrypted-data-in-cassandra)
  - [Moving unencrypted data from a legacy system into Cassandra](#moving-unencrypted-data-from-a-legacy-system-into-cassandra)
  - [Videos](#videos)
  - [References](#references)

## Architecture Overview

### Peer-to-peer architecture

You can load data into one pod in Cassandra, and the other pods should automatically sync through Cassandra’s replication mechanism, depending on your cluster’s configuration. Cassandra uses a peer-to-peer architecture, meaning each node (or pod) can replicate data across the cluster to ensure consistency.
To confirm this setup will work seamlessly:

1. Check the replication factor for your keyspace. This factor determines how many copies of your data exist across the nodes.
2. Ensure the consistency level you plan to use aligns with your data accuracy and availability needs (e.g., QUORUM, ALL).

If your replication factor and consistency level are set correctly, loading data into one pod should indeed propagate it across the others as designed by the Cassandra engine.

### Check the replication factor and consistency settings in Cassandra

To check the replication factor and consistency settings in Cassandra, you can use **CQL (Cassandra Query Language)**. Here’s a step-by-step guide:

1. Connect to Cassandra
    First, open **cqlsh** (Cassandra’s command-line shell) by running:

    ```bash
    cqlsh
    ```

2. View Keyspaces and Replication Factor
    Keyspaces in Cassandra are like databases in other systems, and each keyspace has a replication strategy and factor.

    To list all keyspaces, run:

    ```cql
    DESCRIBE KEYSPACES;
    ```

    Then, to see the details (including replication) for a specific keyspace, run:

    ```cql
    DESCRIBE KEYSPACE your_keyspace_name;
    ```

    Look for something like this in the output:

    ```yaml
    'replication': {
        'class': 'NetworkTopologyStrategy',
        'datacenter1': '3'
    }
    ```

    - **Replication class**:
    - **SimpleStrategy** (used for single data center setups) or **NetworkTopologyStrategy** (used for multi-data center setups).
    - **Replication factor**: This number shows how many copies of data Cassandra keeps across nodes. For example, `'datacenter1': '3'` means three replicas across nodes in that data center.

3. Check Consistency Level
    The **consistency level** isn’t tied to the keyspace but rather specified in each query. Common consistency levels are:
    - **ONE**: Query reads data from one replica.
    - **QUORUM**: Majority of replicas.
    - **ALL**: All replicas must respond.

    You set the consistency level when querying data. Here’s an example:

    ```cql
    CONSISTENCY QUORUM;
    SELECT * FROM your_keyspace.your_table;
    ```

You can find the consistency level in your application’s configuration if you’re querying through a driver (e.g., Java, Python).

> **Quick Summary for Best Practices**
>
> - Set **NetworkTopologyStrategy** with appropriate replication factors for each data center if your setup is multi-data center.
> - Use **QUORUM** for a balanced approach between availability and consistency.

### For a high-performance data migration into Cassandra

For a high-performance data migration into Cassandra, consider these options based on factors like data volume, consistency needs, and cluster configuration:

1. **Use Cassandra Bulk Loader (cassandra-loader or sstableloader)**
   - **sstableloader** is Cassandra's built-in tool for bulk data loading, which is efficient and optimized for large datasets.
   - Steps:
     1. Convert your data into SSTable format.
     2. Use `sstableloader` to load the data into the cluster.
   - **Pros**: Fast, bypasses CQL overhead.
   - **Cons**: Requires extra steps to convert data into SSTable format.

2. **DataStax Bulk Loader (dsbulk)**
   - `dsbulk` is a tool from DataStax designed to simplify bulk data import and export with CSV, JSON, or other formats.
   - Example:
     ```bash
     dsbulk load -k your_keyspace -t your_table -url /path/to/your_data.csv
     ```
   - **Pros**: Optimized, flexible with formats, offers parallelism and batching controls.
   - **Cons**: Additional setup if you don’t already use DataStax tools.

3. **Cassandra Drivers (Java, Python, etc.)**
   - Using official drivers (Java, Python, etc.), you can implement parallel batch processing directly in your code.
   - Configure drivers for **batching** and **asynchronous execution**.
   - **Pros**: Fine-grained control, ability to manage backpressure.
   - **Cons**: Adds coding complexity, may be slower than bulk loaders for extremely large datasets.

4. **Apache Kafka Integration**
   - Using **Kafka Connect** with Cassandra Sink connector, you can stream data from a Kafka topic into Cassandra.
   - **Pros**: Ideal for real-time data migration or integration, can handle large volumes gradually.
   - **Cons**: Requires Kafka setup, not ideal if you’re aiming for a one-time bulk migration.

5. **Import CSV Files (via cqlsh)**
   - `COPY` command in **cqlsh** can import CSV data directly into Cassandra tables:
     ```cql
     COPY your_keyspace.your_table (column1, column2) FROM '/path/to/your_data.csv' WITH HEADER = TRUE;
     ```
   - **Pros**: Simple and quick for small to medium datasets.
   - **Cons**: Limited for very large datasets, lacks parallel processing capabilities.

Recommendations for High Performance
- **Choose `sstableloader` or `dsbulk`** for bulk loading due to their performance and Cassandra compatibility.
- **Partition the load across multiple nodes** if possible to reduce bottlenecks.
- **Adjust the write consistency level** (e.g., use `QUORUM` instead of `ALL`) to balance speed and reliability.
- **Monitor and tweak Cassandra settings** (e.g., compaction, memtable flush thresholds) to optimize performance during the load.

Each method has its strengths, so choosing depends on the volume and frequency of your migration needs. Let me know if you want more detail on any of these approaches!

### Tools for importing and exporting data

Apache Cassandra has several tools that can be used for importing and exporting data. Here are a few common ones:

1. **CQLSH (Cassandra Query Language Shell)**:
   - You can use `COPY` commands in CQLSH to import and export data.
     - **Exporting data**: `COPY keyspace_name.table_name TO 'file.csv';`
     - **Importing data**: `COPY keyspace_name.table_name FROM 'file.csv';`
   - It works well for smaller datasets, but for large datasets, performance can be an issue.

2. **SSTableloader**:
   - This tool is used for bulk importing data directly into Cassandra tables.
   - It can stream the data into the cluster, ensuring it's evenly distributed across nodes.
   - Best for importing large datasets or moving data from one Cassandra cluster to another.

3. **DataStax Bulk Loader (DSBulk)**:
   - DSBulk is a more advanced tool provided by DataStax that allows efficient import and export of data at scale.
     - **Export**: `dsbulk unload -k keyspace_name -t table_name -url file_path`
     - **Import**: `dsbulk load -k keyspace_name -t table_name -url file_path`
   - Supports CSV, JSON, and other formats, and it’s highly optimized for handling large amounts of data.

4. **Apache Spark with Cassandra Connector**:
   - If you have more complex data processing needs or need to import/export data across multiple systems, you can use Apache Spark with the Cassandra Connector to move data to and from Cassandra.

Each of these tools has its use case, with CQLSH being easy for small-scale operations and tools like SSTableloader or DSBulk better suited for larger datasets.

Yes, there are several tools and approaches to **import/export data** from CSV files to **Cassandra**. Some popular ones include:

1. **Cassandra Query Language Shell (CQLSH)**
   - **CQLSH** provides a built-in command called `COPY`, which allows you to import/export data from/to CSV files.
   - **Importing data** from a CSV file into Cassandra:
     ```bash
     COPY keyspace_name.table_name (column1, column2, ...) FROM 'file_path.csv' WITH HEADER = TRUE;
     ```
   - **Exporting data** to a CSV file:
     ```bash
     COPY keyspace_name.table_name (column1, column2, ...) TO 'file_path.csv' WITH HEADER = TRUE;
     ```

   **Pros**: It's simple to use directly from the command line without needing additional tools.
   **Cons**: Limited for handling large datasets, as it might run into performance issues or timeouts.

2. **DataStax Bulk Loader (DSBulk)**
   - **DSBulk** is a more robust tool designed by **DataStax** for high-performance bulk data loading and unloading in and out of Cassandra.
   - It supports CSV files and can handle very large datasets more efficiently than CQLSH.
   - **Importing data** from CSV to Cassandra:
     ```bash
     dsbulk load -k keyspace_name -t table_name -url file_path.csv -header true
     ```
   - **Exporting data** to CSV:
     ```bash
     dsbulk unload -k keyspace_name -t table_name -url file_path.csv -header true
     ```

   **Pros**: Handles large datasets efficiently with more control over batching and parallelism.
   **Cons**: Requires installation of the DSBulk tool, but it’s relatively easy to set up.

3. **Spark with Cassandra Connector**
   - If you are dealing with large-scale data processing, you can use **Apache Spark** with the **Cassandra Connector** to read/write data between CSV and Cassandra.
   - You can load a CSV file into a Spark DataFrame, process it, and then write it to a Cassandra table.

   **Pros**: Powerful for data transformations and large-scale operations.
   **Cons**: Requires setup of Spark and is overkill for smaller or simpler tasks.

4. **Third-party Tools**
   - There are other third-party tools like **Talend**, **Pentaho**, or **Apache Nifi** that can be used to build ETL (Extract, Transform, Load) pipelines, including reading from CSV files and writing to Cassandra.
   - These tools offer drag-and-drop interfaces to build complex workflows for data integration.

   **Pros**: Great for building comprehensive ETL processes.
   **Cons**: Requires installation and configuration of additional software.

If you are looking for an easy option for small to medium data, **CQLSH COPY** and **DSBulk** would be the best choices. For larger datasets or complex data transformations, Spark or third-party ETL tools may be more suitable.

### To upload data into a Cassandra database after generating stage tables

To upload data into a **Cassandra** database after generating stage tables using a PL/SQL procedure, there are several alternatives you can consider for efficiently loading the data into Cassandra. These alternatives can range from tools that handle bulk loading to writing custom scripts. Here are some of the most effective options:

1. **Cassandra Query Language Shell (CQLSH) - COPY Command**
   - If your stage tables can be exported as **CSV files**, you can use the `COPY` command in **CQLSH** to import CSV data into Cassandra. This is one of the simplest methods to load data.
   - **Steps**:
     1. Export the PL/SQL stage tables to CSV format.
     2. Use `COPY` in CQLSH to load the data:
        ```bash
        COPY keyspace.table (column1, column2, ...) FROM 'file_path.csv' WITH HEADER = TRUE;
        ```
   - **Limitations**: It may not be ideal for large datasets due to performance constraints.

2. **DataStax Bulk Loader (DSBulk)**
   - **DSBulk** is a high-performance tool designed specifically for importing/exporting data into/from Cassandra. It handles large datasets more efficiently than the `COPY` command.
   - **Steps**:
     1. Export the PL/SQL tables to CSV.
     2. Use **DSBulk** to load the CSV files into Cassandra:
        ```bash
        dsbulk load -k keyspace_name -t table_name -url file_path.csv -header true
        ```
   - **Advantages**: Can handle parallel processing, batching, and large datasets better than CQLSH.

3. **Apache Spark with Cassandra Connector**
   - For complex transformations or large-scale data migration, you can use **Apache Spark** with the **Cassandra Connector**. Spark can load the data from your staging tables and write it directly to Cassandra.
   - **Steps**:
     1. Use a **Spark job** to read the data from your stage tables (via JDBC, for example).
     2. Transform the data in Spark if necessary.
     3. Use the Cassandra connector to write the transformed data to Cassandra.
     - Example in Scala or Python using the Cassandra connector:
       ```scala
       val df = spark.read.format("jdbc").option("url", "jdbc:mysql://your_url").load()
       df.write.format("org.apache.spark.sql.cassandra").options(Map("keyspace" -> "keyspace_name", "table" -> "table_name")).save()
       ```

4. **ETL Tools (Talend, Pentaho, Apache Nifi)**
   - These are comprehensive ETL (Extract, Transform, Load) tools that can automate the process of reading from your relational stage tables and writing to Cassandra.
   - **Steps**:
     1. Configure a connection to your Oracle/PLSQL database to read the stage tables.
     2. Configure a connection to Cassandra using the tool's Cassandra connector.
     3. Build an ETL job to move the data from the stage tables to Cassandra.

   - **Advantages**: These tools offer user-friendly interfaces and can handle complex data flows and transformations. They are also scalable for large datasets.
   - **Limitations**: They may require additional setup and have a learning curve.

5. **Custom Scripts Using Cassandra Drivers**
   - You can also write custom scripts in languages like **Java**, **Python**, or **Node.js** using the appropriate **Cassandra drivers**.
   - **Steps**:
     1. Use your preferred language to connect to the Oracle database and fetch the data from the stage tables.
     2. Use the corresponding Cassandra driver (e.g., **Datastax Java Driver**, **Python Cassandra Driver**) to insert data into Cassandra in batches.
     - Example in Python using `cassandra-driver`:
       ```python
       from cassandra.cluster import Cluster
       cluster = Cluster(['cassandra_host'])
       session = cluster.connect('keyspace_name')

       query = "INSERT INTO table_name (column1, column2) VALUES (%s, %s)"
       session.execute(query, (value1, value2))
       ```

   - **Advantages**: Full control over the process and can handle custom logic for data migration.
   - **Limitations**: Requires development time and can be complex to manage for large datasets.

6. **Kafka and Cassandra Integration**
   - If you're dealing with streaming data or need a scalable, real-time data pipeline, using **Apache Kafka** as an intermediary between the stage tables and Cassandra could be an option.
   - **Steps**:
     1. Export data from PL/SQL to Kafka topics.
     2. Use a **Kafka consumer** to write data to Cassandra.

   - **Advantages**: This method is highly scalable and can handle real-time data migration.
   - **Limitations**: It’s more complex to set up and configure compared to the other options.

**Conclusion:**
The best choice depends on the volume of data, the complexity of the migration, and whether you need real-time processing or batch loading. For simple and small-to-medium datasets, **CQLSH** or **DSBulk** is ideal. For more complex or large-scale migrations, **Apache Spark**, **ETL tools**, or **custom scripts** are better suited.

## CQLSH (Cassandra Query Language Shell):

- You can use COPY commands in CQLSH to import and export data.
  - Exporting data: COPY keyspace_name.table_name TO 'file.csv';
  - Importing data: COPY keyspace_name.table_name FROM 'file.csv';
- It works well for smaller datasets, but for large datasets, performance can be an issue.

### Export data from a table in your Cassandra container

**To export data from the `student` table in your Cassandra container to a CSV file accessible from your Windows `C:\` directory, follow these steps:**

**Step 1:** Use `COPY` Command in Cassandra

1. Start by entering the Cassandra container and launching `cqlsh`:

   ```bash
   docker exec -it <container_name> cqlsh
   ```

2. In `cqlsh`, use the `COPY` command to export the data:

   ```sql
   COPY test.student (id, name, city) TO '/data/student.csv' WITH HEADER = TRUE;
   ```

   This command creates a file `student.csv` in the `/data` directory of the container.

**Step 2:** Mount the `C:\` Directory to Access the CSV

To make the CSV file accessible on your Windows machine, you need to have a directory in your `C:\` drive mounted to the container. For example, if you want to map `C:\cassandra_data` to `/data` inside the container, you can start the container with:

   ```bash
   docker run -d --name cassandra_container -v C:\cassandra_data:/data cassandra
   ```

   If the container is already running, you’ll need to recreate it with the volume flag, or alternatively, copy the CSV file from the container to your host system after running the `COPY` command.

**Step 3:** Verify the File in Windows

1. After the `COPY` command completes, check `C:\cassandra_data` in your Windows file explorer.
2. You should find `student.csv` there with your exported data.

This method allows you to access the CSV export directly on your Windows file system.

### Error indicates that the `/data` directory doesn't exist

**The error indicates that the `/data` directory doesn't exist inside the Cassandra container. You’ll need to create the directory first, then retry the `COPY` command. Here’s how to do it:**

**Step 1:** Create the `/data` Directory Inside the Container

1. **Open a shell in the Cassandra container**:

   ```bash
   docker exec -it <container_name> bash
   ```

2. **Create the `/data` directory**:

   ```bash
   mkdir /data
   ```

3. **Exit the container shell**:

   ```bash
   exit
   ```

**Step 2:** Run the `COPY` Command Again in `cqlsh`

Now, go back to `cqlsh` and try running the `COPY` command again:

```sql
COPY test.student (id, name, city) TO '/data/student.csv' WITH HEADER = TRUE;

COPY test.student (id, name, city, fees, contact) FROM '/data/student2.csv' WITH HEADER = TRUE;

select * from test.student;
```

**Step 3:** Map `/data` to a Local Windows Directory (Optional)

If you want to make this CSV file accessible in a specific directory on your Windows system (e.g., `C:\cassandra_data`), you need to mount this directory when you start the container:

1. **Stop and remove the current container** (if needed):

   ```bash
   docker stop <container_name>
   docker rm <container_name>
   ```

2. **Start the container with volume mapping**:

   ```bash
   docker run -d --name <container_name> -v C:\cassandra_data:/data cassandra
   ```

After running the `COPY` command, you should find `student.csv` in `C:\cassandra_data` on your Windows system.

### Copy the File from the Container to the Host

You can use a similar approach to copy files from a Docker container to your host system without stopping or removing the container. You can use Docker’s `cp` command to copy files directly from the container to your local machine. Here’s how:

**Step 1:** Copy the File from the Container to the Host

Suppose you want to copy the `student.csv` file from `/data` in the Cassandra container to a directory on your host machine (e.g., `C:\cassandra_data`).

1. Run the `docker cp` command:

   ```bash
   docker cp <container_name>:/data/student.csv C:\cassandra_data\student.csv
   ```

   - `<container_name>`: Replace this with the name or ID of your Cassandra container.
   - `:/data/student.csv`: The path inside the container where the file is located.
   - `C:\cassandra_data\student.csv`: The destination path on your host machine.

This command will copy `student.csv` from the container to the specified directory on your Windows host.

**Example of the Complete Workflow:**

1. **Export the data to a CSV file** inside the container using `cqlsh`:

   ```sql
   COPY test.student (id, name, city) TO '/data/student.csv' WITH HEADER = TRUE;
   ```

2. **Copy the CSV file to your host system**:

   ```bash
   docker cp <container_name>:/data/student.csv C:\cassandra_data\student.csv

   docker cp C:\cassandra_data\student2.csv cassandra:/data/student2.csv
   ```

After this, `student.csv` should be available on your Windows machine under `C:\cassandra_data`. This method is straightforward and doesn’t require mounting directories or restarting containers.

### COPY command in cqlsh to export each table

Yes, if you don’t have the `dsbulk` tool available, you can still use the `COPY` command in `cqlsh` to export each table in a keyspace to individual files. The `COPY` command is useful for smaller data volumes or one-time exports, though it might be slower than `dsbulk` for larger datasets.

Here’s how you can proceed:

Step 1: Connect to `cqlsh`
First, open `cqlsh` and connect to your Cassandra database:

```bash
cqlsh <host_ip> -u <username> -p <password>
```

Step 2: Identify the Tables in Your Keyspace
Run this command to list all tables in your keyspace:

```sql
DESCRIBE TABLES <keyspace_name>;
```

Step 3: Export Each Table Using `COPY`
For each table, use the `COPY` command to export data to a CSV file. Here’s the basic syntax:

```sql
COPY <keyspace_name>.<table_name> TO '<file_path>/<table_name>.csv' WITH HEADER = TRUE;
```

- **`<keyspace_name>`**: Replace with the name of your keyspace.
- **`<table_name>`**: Replace with the name of each table in the keyspace.
- **`<file_path>`**: Specify the directory path where you want to store the exported files.

For example, if you have a table called `customers` in the keyspace `sales`, and you want to export it to `/home/wallace/exports`, use:

```sql
COPY sales.customers TO '/home/wallace/exports/customers.csv' WITH HEADER = TRUE;
```
1. Automating the Process
If you want to export all tables in a keyspace automatically, you can use a script to run `cqlsh` commands for each table. Here’s an example of a Bash script to automate this:

```bash
#!/bin/bash

KEYSPACE="<keyspace_name>"
EXPORT_DIR="/home/wallace/exports"
HOST="<host_ip>"
USERNAME="<username>"
PASSWORD="<password>"

# Make export directory if it doesn’t exist
mkdir -p "$EXPORT_DIR"

# Get list of tables
TABLES=$(cqlsh $HOST -u $USERNAME -p $PASSWORD -e "DESCRIBE TABLES $KEYSPACE;")

# Export each table to CSV
for TABLE in $TABLES; do
    cqlsh $HOST -u $USERNAME -p $PASSWORD -e "COPY $KEYSPACE.$TABLE TO '$EXPORT_DIR/$TABLE.csv' WITH HEADER = TRUE;"
    echo "Exported $TABLE to $EXPORT_DIR/$TABLE.csv"
done
```

This script connects to `cqlsh`, retrieves the list of tables, and exports each table to a CSV file in the specified directory.

### make it executable,

To use the script, save it as a `.sh` file, make it executable, and then run it from the command line. Here’s a step-by-step guide:

1. Save the Script to a File
Open a text editor, paste the script, and save it. For example, save it as `export_cassandra_tables.sh` in your home directory.

2. Update the Script with Your Details
Replace the placeholders in the script with your actual details:

- **`<keyspace_name>`**: The name of the keyspace you want to export.
- **`<host_ip>`**: The IP address or hostname of your Cassandra instance.
- **`<username>`** and **`<password>`**: Your Cassandra login credentials.

For example:

```bash
KEYSPACE="my_keyspace"
EXPORT_DIR="/home/wallace/exports"
HOST="127.0.0.1"
USERNAME="my_user"
PASSWORD="my_password"
```

3. Make the Script Executable
In the terminal, navigate to the directory where you saved the script (e.g., `/home/wallace`) and make it executable:

```bash
chmod +x export_cassandra_tables.sh
```

4. Run the Script
Now, execute the script by running:

```bash
./export_cassandra_tables.sh
```

5. Check the Exported Files
After the script completes, check the directory you specified in `EXPORT_DIR`. Each table in the keyspace should have its own `.csv` file with the data.

Notes:
- Ensure you have permissions to write to the specified `EXPORT_DIR`.
- The script will print a message for each table it exports, so you can monitor the progress.

Let me know if you run into any issues!

### Editor text

On a typical Linux environment, you can use several text editors directly in the terminal. Here are some of the common ones:

1. **nano** (Beginner-Friendly)
   - `nano` is user-friendly, straightforward, and perfect for quick edits.
   - To open a new file or edit an existing one, use:
     ```bash
     nano export_cassandra_tables.sh
     ```
   - After editing, press `Ctrl + O` to save, then `Enter` to confirm, and `Ctrl + X` to exit.

2. **vim** (Advanced, Powerful)
   - `vim` is a powerful editor but has a steeper learning curve.
   - To edit the script, run:
     ```bash
     vim export_cassandra_tables.sh
     ```
   - In `vim`, press `i` to start editing. After editing, press `Esc`, then type `:wq` and press `Enter` to save and exit.

3. **gedit** (Graphical, if you have a GUI)
   - If you’re on a Linux server with a graphical interface, `gedit` provides a graphical text editor.
   - To open it:
     ```bash
     gedit export_cassandra_tables.sh &
     ```
   - This opens a new window, and you can save changes as you would in any GUI editor.

4. **Other Editors**: 
   - **`vi`**: Similar to `vim`, useful if `vim` isn’t installed.
   - **`emacs`**: Powerful, though usually requires installation.

If unsure, start with `nano` since it’s available on most Linux systems and is easy to use. Let me know if you need guidance with any specific editor!

## DataStax Bulk Loader (DSBulk)

**What is DSBulk?**

- A powerful tool for moving data to/from Cassandra and files in the system.
- Supports CSV, JSON, and other formats, and it’s highly optimized for handling large amounts of data.
- Operates via a command-line interface.
- Designed as a "Customer First" feature for DataStax Enterprise (DSE) users.

DSBulk is essentially an advanced, high-performance import tool specifically built for DSE, making it an ideal fit for our data migration project. This command-line interface tool simplifies bulk data operations, making it efficient for scheduled jobs, such as cron jobs. It was created with the intent to streamline the process of getting data in and out of the database, overcoming the limitations of previous tools.

- Export: dsbulk unload -k keyspace_name -t table_name -url file_path
- Import: dsbulk load -k keyspace_name -t table_name -url file_path

**Why DSBulk?**

- Loading large volumes of data into DSE has traditionally been challenging.
- Unloading data was also a requirement.
- Previous tools had limitations:
  - `CQLSH COPY FROM` was not performant or robust for large datasets.
  - `SSTableLoader` required data in SSTable format, complicating data transfer.
  - `cassandra loader` was not formally supported.

**DSBulk Use Cases**

- **Loading Data from Multiple Files:** Often, data from sources may be organized by date or batch and delivered as multiple files. DSBulk makes it easy to load these “piles” of files into the database efficiently.

- **One-Time Loads and Production Flow:** DSBulk is excellent for one-time loads, such as bringing data into production, or incorporating it as part of a regular data pipeline to ensure smooth data flow.

- **Initial Developer Experience:** When developers need a quick start by loading data from familiar sources, DSBulk can streamline this by importing data from an existing data source into our current environment.

- **Backup Capabilities:** For a quick backup, DSBulk can unload data from a table into a file that can be stored. This is particularly useful for static or reference data that may need occasional backups without complex solutions.

- **Migration or Data Model Changes:** When there’s a need to change the data model, DSBulk simplifies migration. It allows unloading from the current data model and reloading into a new structure, supporting flexible data evolution.

Overall, DSBulk solves many of these issues and is especially useful when working with large data volumes, which would be less efficient with tools like the `COPY` command or `SSTableLoader`.

### How to install

- **Step 1: Download DataStax Bulk Loader**
  - [DataStax Bulk Loader](https://downloads.datastax.com/#bulk-loader)
  - [GitHun datastax/dsbulk](https://github.com/datastax/dsbulk)

- **Step 2: Unzip, check %JAVA_HOME% and add into PATH**
  - Unzip the file `dsbulk-1.11.0.zip`
  - In cmd prompt, type `echo %JAVA_HOME%` and see if it is installed
  - Add `c:\dsbulk-1.11.0\bin` in the PATH

- **Step 3: Test the load/unload**

   ```cmd
   dsbulk load -k test -t student -url C:\\cassandra_data\\student_import.csv

   dsbulk unload -k test -t student -url C:\\cassandra_data\\student_export.csv
   ```

- **Step 4: More advanced**
  - [Introduction and Loading](https://www.datastax.com/blog/datastax-bulk-loader-introduction-and-loading)
  - [Data Samples](https://gist.github.com/brianmhess/8864cf0cb0ce9ea1fd64e579e9f41100)

   ```cmd
   dsbulk load -url C:\\cassandra_load\\tmp\\dsbulkblog\\iris.csv -k dsbulkblog -t iris_with_id

   dsbulk load -url C:\\cassandra_load\\tmp\\dsbulkblog\\sportsteams.csv -k dsbulkblog -t categories_list

   dsbulk unload -url C:\\cassandra_load\\tmp\\unload -k dsbulkblog -t iris_with_id
   ```

### the `datastax/dsbulk` image

The error you’re seeing indicates that the `datastax/dsbulk` image is not available in the Docker registry, which is likely because an official `dsbulk` image doesn’t exist. Instead, we can install `dsbulk` locally and run it on your host system, or create a custom Docker image for it.

Here are both options:

**Option 1:** Install and Use `dsbulk` Locally

1. **Download dsbulk** from the [DataStax Bulk Loader GitHub page](https://github.com/datastax/dsbulk).

2. **Extract the downloaded file** and navigate to the `dsbulk` directory.

3. Run `dsbulk` commands directly from your command line (you'll need to point it to your Cassandra container using the host IP or container name).

   Here’s an example command to load data from `C:\cassandra_data\student_import.csv` into the `student` table:

   ```bash
   dsbulk load -h <Cassandra_host_name_or_IP> -k test -t student -url C:\cassandra_data\student_import.csv -b
   ```

**Option 2:** Create a Custom Docker Image for `dsbulk`

If you’d prefer to keep it all within Docker, here’s how to create a simple Docker image for `dsbulk`:

1. **Create a Dockerfile** to set up `dsbulk`:

   In a new directory, create a file named `Dockerfile` with the following contents:

   ```Dockerfile
   FROM openjdk:8-jre
   RUN apt-get update && apt-get install -y wget unzip
   RUN wget https://github.com/datastax/dsbulk/releases/download/v1.7.0/dsbulk-1.7.0.tar.gz
   RUN tar -xzf dsbulk-1.7.0.tar.gz && mv dsbulk-1.7.0 /opt/dsbulk
   ENV PATH="/opt/dsbulk/bin:$PATH"
   ```

2. **Build the Docker image**:

   ```bash
   docker build -t custom-dsbulk .
   ```

3. **Run dsbulk commands** using the custom image:

   ```bash
   docker run --rm -v C:\cassandra_data:/data --network cassandra custom-dsbulk load \
     -h <Cassandra_host_name_or_IP> \
     -k test \
     -t student \
     -url /data/student_import.csv \
     -b
   ```

This custom image setup allows you to use `dsbulk` within Docker similarly to how it’s described in the quick start guides, but with the flexibility to run it as needed without requiring a pre-made image.

### Confirm Port Accessibility

It looks like your Cassandra container is up, with port `9042` mapped. However, if `dsbulk` is still not connecting, here are a couple of troubleshooting steps:

1. **Confirm Port Accessibility**: Sometimes, even if a container shows as "Up," ports may not be accessible on `localhost` or `127.0.0.1`. Try running:
   ```bash
   telnet 127.0.0.1 9042
   ```
   or, on Linux:
   ```bash
   nc -zv 127.0.0.1 9042
   ```
   If the port is accessible, you should see a connection confirmation. If not, double-check your Docker network settings.

2. **Restart the Cassandra Container**: Occasionally, Cassandra requires a restart to establish proper connections on all ports. Run:
   ```bash
   docker restart cassandra
   ```

3. **Try Connecting with cqlsh**: This will verify if Cassandra on Docker is reachable. Run:
   ```bash
   docker exec -it cassandra cqlsh 127.0.0.1
   ```
   If `cqlsh` connects successfully, your Cassandra setup is working, and `dsbulk` should also be able to connect.

Once you’ve confirmed access, retry `dsbulk`:
```bash
dsbulk load -h 127.0.0.1 -k test -t student -url C:\\cassandra_data\\student_import.csv
```

The port used by DSBulk to connect to Cassandra is specified by the -port option in the DSBulk command. For example, to specify port 9876 for the cluster hosts, you would use the following command:

`dsbulk load -url ~/export.csv -k ks1 -t table1 -h '10.200.1.3, 10.200.1.4' -header false -m '0=col1,1=col3' -port 9876 `

Here are some other ports used by Cassandra: 
- 7000: Default port for cluster communication
- 7001: Port for SSL-enabled cluster communication
- 9042: Port for native protocol clients
- 7199: Port for JMX 

You can configure the internode communication and native protocol ports in the cassandra-yaml file. The JMX port can be configured in cassandra-env.sh.

Let me know if you encounter any further issues!

### DSBulk in your user folder

Yes, you can definitely install DSBulk in your user folder without needing `sudo` permissions. Here’s how:

1. **Download DSBulk**:
   Use `curl` or `wget` to download DSBulk to your user directory.

   ```bash
   curl -L -o ~/dsbulk.tar.gz "https://downloads.datastax.com/dsbulk/dsbulk-1.8.0.tar.gz"
   ```

2. **Extract the Archive**:
   Extract the contents in your home directory or a subdirectory like `~/bin`.

   ```bash
   mkdir -p ~/bin
   tar -xzf ~/dsbulk.tar.gz -C ~/bin
   ```

3. **Rename the Directory for Simplicity**:
   Rename the extracted folder if necessary (it might be named something like `dsbulk-1.8.0`).

   ```bash
   mv ~/bin/dsbulk-1.8.0 ~/bin/dsbulk
   ```

4. **Add DSBulk to Your PATH**:
   To make DSBulk accessible from any terminal, add it to your PATH by editing your `.bashrc` (or `.bash_profile`) file.

   ```bash
   echo 'export PATH=$PATH:$HOME/bin/dsbulk/bin' >> ~/.bashrc
   source ~/.bashrc
   ```

5. **Verify the Installation**:
   Now check if DSBulk is set up correctly.

   ```bash
   dsbulk --version
   ```

This approach should allow you to run DSBulk without needing root permissions. Let me know if you have any issues!

### check if the ports are available

To check if the ports are available on the host machine for communication outside the container, you can follow these steps:

1. **Use `netstat` or `ss`**:
   You can check which ports are open and listening on your host machine by using `netstat` or `ss`. This will show if the container's port is accessible from the outside.

   ```bash
   # Using netstat
   netstat -tuln | grep <port_number>

   # Using ss (if netstat is not available)
   ss -tuln | grep <port_number>
   ```

   Look for the line showing the port number you’re interested in (replace `<port_number>` with the actual port number). If you see a `LISTEN` status with `0.0.0.0:<port_number>`, it means the port is open to any IP address. If it's bound to `127.0.0.1`, it’s only accessible locally.

2. **Check Docker Port Mapping**:
   If the application runs in Docker, confirm that the container port is mapped to the host port by running:

   ```bash
   docker ps
   ```

   Look at the `PORTS` column for the relevant container, which should show something like `0.0.0.0:<host_port>-><container_port>/tcp`. This indicates that the container's port is mapped to the specified host port.

3. **Test Connectivity with `telnet` or `nc`**:
   From another machine or a different terminal, you can check if the host port is reachable:

   ```bash
   # Using telnet
   telnet <host_ip> <port_number>

   # Using netcat (nc)
   nc -zv <host_ip> <port_number>
   ```

   If the port is accessible, it will connect; otherwise, you’ll see a "Connection refused" or "Timed out" message.

4. **Use `curl` or `wget` for HTTP Ports**:
   If the port is used for an HTTP service, you can use `curl` to test if it’s accessible:

   ```bash
   curl http://<host_ip>:<port_number>
   ```

This should confirm if the port is available on the host machine and accessible from outside the container. Let me know if you need further assistance with any of these steps!

### Check Kubernetes ports

Since the host machine is a jump server running Kubernetes, you can check if the ports are exposed outside the Kubernetes cluster by examining the services configured in Kubernetes. Here are some methods to verify port availability:

1. **Check Kubernetes Services**
   Run the following command to see the services running in Kubernetes and their exposed ports:

   ```bash
   kubectl get services -o wide
   ```

   This will show information about each service, including the `EXTERNAL-IP` (if it’s accessible externally) and the `PORT(S)`. Look for services with a type `NodePort` or `LoadBalancer`, as these are designed to be accessible from outside the cluster.

   - **ClusterIP**: Only accessible within the cluster.
   - **NodePort**: Exposes the service on a specific port on each node in the cluster.
   - **LoadBalancer**: Exposes the service externally, often used in cloud environments.

2. **Verify NodePort Accessibility (for NodePort Services)**
   If a service is of type `NodePort`, the port should be accessible on the host machine's IP and specified port range (usually between 30000-32767).

   Use `nc` or `telnet` to test the port from the jump server:

   ```bash
   nc -zv <host_ip> <NodePort>
   ```

3. **Check LoadBalancer IP or Ingress (for LoadBalancer Services)**
   If you are using a LoadBalancer service, it might have an external IP. You can test access to that IP with `curl` or `nc`, depending on the service’s protocol:

   ```bash
   curl http://<external_ip>:<port>
   ```

4. **Use `kubectl port-forward` for Local Access**
   If the service is only accessible within the Kubernetes cluster and you need to test it locally on the jump server, you can use `kubectl port-forward`:

   ```bash
   kubectl port-forward svc/<service_name> <local_port>:<service_port>
   ```

   This will forward traffic from a local port on the jump server to the Kubernetes service port.

These methods should help you verify if the necessary ports are accessible on the jump server and how they’re mapped within Kubernetes. Let me know if you need further assistance with these steps!

## Use Cassandra Bulk Loader (cassandra-loader or sstableloader) [[1]](https://cassandra.apache.org/doc/stable/cassandra/operating/bulk_loading.html)

- sstableloader is Cassandra's built-in tool for bulk data loading, which is efficient and optimized for large datasets.
  - Convert your data into SSTable format.
  - Use sstableloader to load the data into the cluster.
- Fast, bypasses CQL overhead and requires extra steps to convert data into SSTable format.

## Spring Batch and Spring Data for Apache Cassandra

You can use **Spring Batch** to insert data directly into Apache Cassandra by integrating **Spring Data Cassandra** into your batch processing job. Here's an outline of how you can achieve this:

### Steps to Insert Data into Cassandra using Spring Batch

1. **Add Dependencies**:
   Include the necessary dependencies for Spring Batch and Spring Data Cassandra in your `pom.xml` (for Maven) or `build.gradle` (for Gradle).

   **Maven Example**:
   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-batch</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-cassandra</artifactId>
   </dependency>
   ```

2. **Configure Cassandra**:
   Set up your Cassandra connection properties in the `application.properties` or `application.yml` file:
   
   ```properties
   spring.data.cassandra.contact-points=localhost
   spring.data.cassandra.port=9042
   spring.data.cassandra.keyspace-name=mykeyspace
   spring.data.cassandra.username=cassandra
   spring.data.cassandra.password=cassandra
   ```

3. **Create an Entity Class**:
   Define your Cassandra entity that represents the table where the data will be inserted.

   ```java
   @Table("my_table")
   public class MyEntity {
       @PrimaryKey
       private String id;

       private String name;

       // Getters and Setters
   }
   ```

4. **Set up a Repository**:
   Use Spring Data Cassandra’s repository pattern to interact with the database. Create a repository interface to insert data into Cassandra.

   ```java
   public interface MyEntityRepository extends CassandraRepository<MyEntity, String> {
   }
   ```

5. **Create the Spring Batch Job**:
   Define a Spring Batch job that will read data (from a CSV, database, or any other source) and write it to Cassandra using the repository.

   - **Reader**: Can be any Spring Batch `ItemReader`, such as `FlatFileItemReader` (for reading from CSV) or `JdbcCursorItemReader` (for reading from a relational database).
   - **Writer**: The `ItemWriter` will use your Cassandra repository to insert data.

   Example:

   ```java
   @Configuration
   public class BatchConfig {

       @Autowired
       private MyEntityRepository myEntityRepository;

       @Bean
       public Job importJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory, ItemReader<MyEntity> reader) {
           Step step = stepBuilderFactory.get("step")
                   .<MyEntity, MyEntity>chunk(100)
                   .reader(reader)
                   .writer(writer())
                   .build();

           return jobBuilderFactory.get("importJob")
                   .start(step)
                   .build();
       }

       @Bean
       public ItemWriter<MyEntity> writer() {
           return new ItemWriter<MyEntity>() {
               @Override
               public void write(List<? extends MyEntity> items) throws Exception {
                   myEntityRepository.saveAll(items);
               }
           };
       }

       // Reader definition here (could be FlatFileItemReader or any other reader)
   }
   ```

6. **Run the Job**:
   You can now run your Spring Batch job, and the data will be written directly into the Cassandra database.

### Key Points:
- **ItemReader**: Reads data from any source (file, DB, etc.).
- **ItemProcessor** (Optional): You can transform the data if needed.
- **ItemWriter**: Writes data to Cassandra using the `CassandraRepository`.
- Spring Data Cassandra handles all the low-level communication with Cassandra for you.

This approach leverages both Spring Batch’s batch processing capabilities and Spring Data Cassandra’s seamless interaction with the Cassandra database, allowing you to efficiently insert data in a batch-oriented way.

## Kafka and Cassandra Integration: Apache Kafka as an intermediary between the stage tables and Cassandra could be an option.

## Decrypt your encrypted data in Cassandra

To decrypt your encrypted data in **Cassandra** using the `ciphertextkey` field, you’ll need to:

1. **Read the encrypted data and the `ciphertextkey` from the table**.
2. **Use the appropriate decryption algorithm** (e.g., AES, RSA) along with the key to decrypt the encrypted data.

**Steps to Decrypt Encrypted Data in Cassandra Using Spring Batch**

Assuming the data encryption algorithm is known, and you have the `ciphertextkey` stored for each record, here's how you can integrate decryption into your Spring Batch job.

**Step 1: Fetch Encrypted Data**:
First, read the encrypted data from Cassandra, which includes both the `ciphertextkey` and the encrypted data field.

Example of an entity class that represents the encrypted data and the `ciphertextkey`:

```java
@Table("my_table")
public class MyEncryptedEntity {

    @PrimaryKey
    private String id;

    private String encryptedData;

    private String ciphertextkey;

    // Getters and Setters
}
```

**Step 2: Create a Decryption Method**:
Implement a method to decrypt the data using the `ciphertextkey` and the encryption algorithm used. For example, if AES encryption was used, you can write a decryption method using Java’s built-in `javax.crypto` package.

Here’s a sample decryption method using AES:

```java
import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

public class DecryptionUtil {

    private static final String ALGORITHM = "AES";

    public static String decrypt(String encryptedData, String ciphertextkey) throws Exception {
        byte[] decodedKey = Base64.getDecoder().decode(ciphertextkey);
        SecretKeySpec secretKey = new SecretKeySpec(decodedKey, ALGORITHM);
        
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, secretKey);

        byte[] encryptedBytes = Base64.getDecoder().decode(encryptedData);
        byte[] decryptedBytes = cipher.doFinal(encryptedBytes);

        return new String(decryptedBytes);
    }
}
```

This method assumes that the `ciphertextkey` is stored in Base64 format and that AES encryption is used. Adjust it as necessary for your encryption algorithm.

**Step 3: Process the Data in Spring Batch**:
In Spring Batch, use an **ItemProcessor** to decrypt the data before writing it to the target location.

Example of a processor that decrypts the data:

```java
import org.springframework.batch.item.ItemProcessor;

public class DecryptionProcessor implements ItemProcessor<MyEncryptedEntity, MyDecryptedEntity> {

    @Override
    public MyDecryptedEntity process(MyEncryptedEntity item) throws Exception {
        // Decrypt the data
        String decryptedData = DecryptionUtil.decrypt(item.getEncryptedData(), item.getCiphertextkey());

        // Map the decrypted data to a new entity
        MyDecryptedEntity decryptedEntity = new MyDecryptedEntity();
        decryptedEntity.setId(item.getId());
        decryptedEntity.setDecryptedData(decryptedData);

        return decryptedEntity;
    }
}
```

In this processor:
- `MyEncryptedEntity` represents the data read from Cassandra, containing encrypted data.
- `MyDecryptedEntity` represents the entity with the decrypted data.

**Step 4: Write the Decrypted Data**:
Once the data is decrypted, you can either store it back in another Cassandra table or process it further. Here’s an example of writing the decrypted data back to a new Cassandra table:

```java
@Table("my_decrypted_table")
public class MyDecryptedEntity {

    @PrimaryKey
    private String id;

    private String decryptedData;

    // Getters and Setters
}
```

Then use the **ItemWriter** in Spring Batch to save the decrypted data:

```java
@Bean
public ItemWriter<MyDecryptedEntity> writer() {
    return new ItemWriter<MyDecryptedEntity>() {
        @Override
        public void write(List<? extends MyDecryptedEntity> items) throws Exception {
            // Assuming MyDecryptedEntityRepository is a CassandraRepository
            myDecryptedEntityRepository.saveAll(items);
        }
    };
}
```

**Step 5: Integrate the Processor into the Spring Batch Job**:

Finally, integrate everything into your batch job configuration:

```java
@Bean
public Job decryptJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory, 
                      ItemReader<MyEncryptedEntity> reader, ItemWriter<MyDecryptedEntity> writer) {

    Step step = stepBuilderFactory.get("decryptStep")
            .<MyEncryptedEntity, MyDecryptedEntity>chunk(100)
            .reader(reader)
            .processor(new DecryptionProcessor())  // Decrypts data
            .writer(writer)  // Writes decrypted data
            .build();

    return jobBuilderFactory.get("decryptJob")
            .start(step)
            .build();
}
```

> **Summary:**
> 
> 1. **Read the encrypted data** from Cassandra using `ItemReader`.
> 2. **Decrypt the data** using an `ItemProcessor`, utilizing a decryption utility like `javax.crypto`.
> 3. **Write the decrypted data** back to Cassandra or any other destination using `ItemWriter`.

This approach works seamlessly in Spring Batch, allowing you to process large volumes of encrypted data in a batch job and securely decrypt them using the corresponding `ciphertextkey` stored in the table.

## Moving unencrypted data from a legacy system into Cassandra

For the purpose of your migration, where you are moving unencrypted data from a legacy system into Cassandra and need to **encrypt the data** before storing it in Cassandra tables, you can follow a process similar to the decryption example, but in reverse. Here’s how to do it:

**Steps to Encrypt Data from Legacy System and Store in Cassandra using Spring Batch**

1. **Extract Data from Legacy System**:
   You can use an **ItemReader** in Spring Batch to read the unencrypted data from the legacy system. This could be a relational database, flat files (CSV), or any other data source.

2. **Encrypt Data**:
   Use an **ItemProcessor** to encrypt the data before it is written into Cassandra. The encryption key (`ciphertextkey`) should be generated or provided as part of the migration process.

3. **Store Encrypted Data in Cassandra**:
   Use an **ItemWriter** to write the encrypted data along with the generated encryption key into Cassandra tables.

**Step 1.** **Encryption Logic**:
Let’s define an encryption utility that uses a strong algorithm like **AES** for encrypting the data. You can customize this based on your encryption needs.

Here’s an example of an encryption method using AES:

```java
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

public class EncryptionUtil {

    private static final String ALGORITHM = "AES";

    public static String encrypt(String data, String secretKey) throws Exception {
        SecretKeySpec secretKeySpec = new SecretKeySpec(Base64.getDecoder().decode(secretKey), ALGORITHM);
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);

        byte[] encryptedBytes = cipher.doFinal(data.getBytes());
        return Base64.getEncoder().encodeToString(encryptedBytes);
    }

    public static String generateSecretKey() throws Exception {
        KeyGenerator keyGen = KeyGenerator.getInstance(ALGORITHM);
        keyGen.init(256); // AES-256
        SecretKey secretKey = keyGen.generateKey();
        return Base64.getEncoder().encodeToString(secretKey.getEncoded());
    }
}
```

- `encrypt()`: Takes plain text data and a secret key, encrypts it, and returns the encrypted data.
- `generateSecretKey()`: Generates a new AES key (you can use this method to generate a unique key per record or a shared key across records).

**Step 2.** **Read Unencrypted Data**:
You will need an **ItemReader** to read the unencrypted data from your legacy system. For example, if the legacy system is a relational database, you could use `JdbcCursorItemReader`.

```java
@Bean
public JdbcCursorItemReader<LegacyEntity> reader(DataSource dataSource) {
    JdbcCursorItemReader<LegacyEntity> reader = new JdbcCursorItemReader<>();
    reader.setDataSource(dataSource);
    reader.setSql("SELECT id, data FROM legacy_table");
    reader.setRowMapper(new LegacyEntityRowMapper());
    return reader;
}
```

Where `LegacyEntity` represents the unencrypted data from the legacy system.

**Step 3.** **Encrypt Data in an ItemProcessor**:
Now, use an **ItemProcessor** to encrypt the data and generate a `ciphertextkey` for each record before writing it to Cassandra.

```java
import org.springframework.batch.item.ItemProcessor;

public class EncryptionProcessor implements ItemProcessor<LegacyEntity, EncryptedEntity> {

    @Override
    public EncryptedEntity process(LegacyEntity item) throws Exception {
        // Generate encryption key
        String secretKey = EncryptionUtil.generateSecretKey();

        // Encrypt the data
        String encryptedData = EncryptionUtil.encrypt(item.getData(), secretKey);

        // Map to the encrypted entity
        EncryptedEntity encryptedEntity = new EncryptedEntity();
        encryptedEntity.setId(item.getId());
        encryptedEntity.setEncryptedData(encryptedData);
        encryptedEntity.setCiphertextkey(secretKey);  // Store the key

        return encryptedEntity;
    }
}
```

- `LegacyEntity`: The entity containing unencrypted data from the legacy system.
- `EncryptedEntity`: The entity that will be written to Cassandra with the encrypted data and `ciphertextkey`.

**Step 4.** **Write Encrypted Data to Cassandra**:
Once the data is encrypted, you can use an **ItemWriter** to store the encrypted data in Cassandra.

```java
@Bean
public ItemWriter<EncryptedEntity> cassandraWriter(MyEncryptedEntityRepository repository) {
    return new ItemWriter<EncryptedEntity>() {
        @Override
        public void write(List<? extends EncryptedEntity> items) throws Exception {
            repository.saveAll(items);
        }
    };
}
```

The `MyEncryptedEntityRepository` should be a **Spring Data Cassandra repository** to handle storing data in Cassandra.

```java
public interface MyEncryptedEntityRepository extends CassandraRepository<EncryptedEntity, String> {
}
```

**Step 5.** **Define the Encrypted Entity Class**:
The encrypted data and the encryption key should be stored in Cassandra. Define a class that represents this:

```java
@Table("encrypted_table")
public class EncryptedEntity {

    @PrimaryKey
    private String id;

    private String encryptedData;

    private String ciphertextkey;

    // Getters and Setters
}
```

**Step 6.** **Configure Spring Batch Job**:
Now, configure the Spring Batch job to perform the migration:

```java
@Bean
public Job migrationJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory, 
                        ItemReader<LegacyEntity> reader, ItemProcessor<LegacyEntity, EncryptedEntity> processor,
                        ItemWriter<EncryptedEntity> writer) {

    Step step = stepBuilderFactory.get("encryptionStep")
            .<LegacyEntity, EncryptedEntity>chunk(100)
            .reader(reader)
            .processor(processor)
            .writer(writer)
            .build();

    return jobBuilderFactory.get("migrationJob")
            .start(step)
            .build();
}
```

> **Summary of the Process:**
>
> 1. **Read unencrypted data** from the legacy system using an **ItemReader**.
> 2. **Encrypt the data** using an **ItemProcessor** that leverages an encryption utility. Also, generate and store the `ciphertextkey`.
> 3. **Write the encrypted data** and the encryption key into Cassandra using an **ItemWriter**.

This process ensures that your data from the legacy system is securely encrypted before being stored in the Cassandra database.

## Videos
 * [DS210.26 DSE DSBulk | Operations with Apache Cassandra](https://www.youtube.com/watch?v=F-JS6Q6iVyI)
	> [<img src="https://img.youtube.com/vi/F-JS6Q6iVyI/0.jpg" width="200">](https://www.youtube.com/watch?v=F-JS6Q6iVyI "DatsStax Enterprise DsBulk utility has a lot of power. It is an awesome import tool, specifically built for DataStax Enterprise. Learn more about DsBulk in this unit. by DataStax Developers 2,608 views 3 minutes, 13 seconds")

## References
1. https://king.host/wiki/artigo/como-liberar-o-uso-do-telnet-no-windows-10/
2. 