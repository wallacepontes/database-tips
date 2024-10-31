# Apache Cassandra Database [[1]](https://www.youtube.com/watch?v=J-cSy5MeMOA)

Apache Cassandra is an open source NoSQL distributed database. This beginner's Cassandra course has four modules. The first three modules will cover the basics Apache Cassandra data modeling. The final module covers practical applications of Cassandra. 

## Course Contents
- [Apache Cassandra Database \[1\]](#apache-cassandra-database-1)
  - [Course Contents](#course-contents)
  - [Module 1: Cassandra Intro and Background](#module-1-cassandra-intro-and-background)
    - [Lesson 1: Apache Cassandra vs. Relational Database](#lesson-1-apache-cassandra-vs-relational-database)
      - [Transactions in Cassandra](#transactions-in-cassandra)
      - [Relational Joins](#relational-joins)
      - [De-normalization](#de-normalization)
      - [Referential Integrity](#referential-integrity)
    - [Lesson 2: Terminology](#lesson-2-terminology)
    - [Lesson 3: SQL Fundamentals](#lesson-3-sql-fundamentals)
    - [Lesson 4: Partitions](#lesson-4-partitions)
  - [Module 2:  Cassandra™ Building Blocks.](#module-2--cassandra-building-blocks)
    - [Lesson 1: De-normalization](#lesson-1-de-normalization)
  - [(31:29) Module 3: Data Modeling](#3129-module-3-data-modeling)
  - [(1:04:22) Module 4: Practical Applications](#10422-module-4-practical-applications)
  - [Video](#video)
  - [References](#references)

## Module 1: Cassandra Intro and Background

This module will cover the basics of Cassandra data model. Upon completion of this module, you will be able to:

- Differentiate between relational databases and an Apache Cassandra database
- Recognize the basic terminologies used in Cassandra tables
- Explore CQL fundamentals
- Identify how partitions, primary keys, and clustering columns are used in Cassandra tables
- Observe the concepts of replication and consistency

### Lesson 1: Apache Cassandra vs. Relational Database

- **Differentiation Overview**

| Apache Cassandra Database | Relational Database|
|-|-|
| Cassandra modeling methodology focused on specific queries | Sample relational model methodology, often focused on flexibility |
| Data structure prioritizes Application - Model - Data | Data structure prioritizes Data - Model - Application |
| Queries are king: data models are built to answer specific queries | Entities are king: schema normalization keeps data generic |
| Primary keys are vital for determining data location in the cluster | Primary keys are mainly used for uniqueness |
| Distributed architecture with no single point of failure | Often have single points of failure |
| CAP theorem: prioritizes Availability and Partition Tolerance | ACID compliance ensures strong consistency |
| De-normalization reduces dependencies between tables | Joins and indexes normalize data across tables |
| Referential integrity is not enforced | Referential integrity is enforced |

#### Transactions in Cassandra

- Apache Cassandra does not support traditional ACID transactions due to the performance impact.
- *CAP Theorem*: Cassandra, by default, is an AP system (Availability and Partition Tolerance) over consistency. However, Cassandra allows users to adjust consistency by setting the number of nodes that must respond to a read or write request.

![CAP Theorem](https://i.pinimg.com/736x/26/bf/c0/26bfc0c642d37d05b16db286b7b6d18f.jpg)


#### Relational Joins

- **Relational Databases**: Use join statements to combine data from different tables, which can impact performance, especially in distributed setups.
- **Cassandra**: Lacks support for joins. Instead, data is de-normalized for each table to store relevant information together for faster access.

#### De-normalization

- In Cassandra, de-normalization is standard practice to ensure that necessary information is contained within a single table, optimizing for the read-heavy workloads typical in Cassandra.
- De-normalization may lead to duplicate data across tables, but it ensures cost-efficiency and fast retrieval.
- Developers design Cassandra tables upfront based on expected queries, unlike relational databases where joins can be used to combine data on the fly.

#### Referential Integrity

- Relational databases enforce referential integrity, requiring values in one table to match values in another.
- In Cassandra, referential integrity is not enforced by the database and should instead be managed by the application logic if necessary.

| Relational Data Modeling | Cassandra Data Modeling |
| --- | --- |
| Enforces referential integrity constraints for joins | Referential integrity is handled by application logic |
| Requires relational constraints for joins | Avoids joins to optimize performance |
| --- | Can enforce referential logic at application level as needed |

### Lesson 2: Terminology

Cassandra uses some database terms with specific meanings in the context of distributed data modeling. Key terms include:

- **Data Model**: Defines how data is structured to optimize for specific queries rather than generalized relationships between entities. It is based on expected query patterns.
- **Keyspace**: The outermost container in Cassandra, holding tables and replication settings.
- **Table**: A collection of rows and columns within a keyspace, optimized around specific query patterns.
- **Partition**: The grouping of rows stored on a node based on a partition key. Partitions allow Cassandra to retrieve data based on predefined keys, enabling fast lookups. Think of it as a roadmap directing data to specific nodes.
- **Primary Key**: Defines data uniqueness and the placement of data within the cluster. It includes the partition key and optionally clustering columns.
- **Partition Key**: The initial part of the primary key determining the node on which data is stored. A strong partitioning strategy helps avoid data hotspots.
- **Columns**: Contain data locally within each table and are referred to as "cells" within rows.
- **Row**: Represents a unique entry in a partition, identified by the primary key.
- **Clustering Column**: Defines the order of data within a partition, enabling complex query patterns within each partition. Data is sorted in ascending order by default but can be customized.

![Terminology](https://editor.analyticsvidhya.com/uploads/37209a.png)

### Lesson 3: SQL Fundamentals 

Cassandra query language or CQL is a nosql language.

- Basic CQL Concepts

| Building Blocks | Basic CQL Commands |
|-|-|
| Keyspaces | SOURCE |
| Tables | ALTER TABLE |
| Primary Keys | TRUNCATE |
| | SELECT |

- **Keyspaces**
  - A top-level container to organize a related set of tables, it is very similar to a relational databases schema.

    ```sql
    CREATE KEYSPACE techframe WITHREPLICATION={'class':'SimpleStrategy', 'replication_factor':1};
    ```

    ![keyspace](https://miro.medium.com/v2/resize:fit:640/format:webp/1*Yiq6XbFEuZiyyJ36sSTTLg.png)

  - Defines replication settings that describe how many copies of a given piece of data are being stored within a cluster. Once you've created a keyspace in cqlsh, you may want to select the keyspace that will be used for your subsequent command.
  - USE switches between keyspaces: `use techframe;`
  - Tables
    - Keyspaces contain tables and tables contain data
    - Every Cassandra table has a primary key

    ```sql
    CREATE TABLE table1(
        column1 TEXT,
        column2 TEXT,
        column3 INT,
        PRIMARY KEY(column1)
    );
    CREATE TABLE users(
        user_id UUID,
        first_name TEXT,
        last_name TEXT,
        PRIMARY KEY(user_id)
    );
    ```

  - Primary Keys clause is a unique identification for each row, it is used to uniquely identify rows. However, a partition key alone is not enough to ensure uniqueness of a row. It is the primary key clause as a whole that determines uniqueness of a row within a partition.
  - SELECT command is used to read data.

    ```sql
    SELECT * 
    FROM table1;

    SELECT column1, column2, column3
    FROM table1;

    SELECT *
    FROM table1
    LIMIT 10;

    SELECT COUNT(*)
    FROM table1;
    ```

  - TRUNCATE command removes data from a table, it sends a JMS command to all nodes to delete sstables that hold the data, if any of these nodes are down the command will fail.
  - ALTER TABLE command makes changes in the table like:
    - Change the data type of a column, 
    - Add columns
    - Drop columns
    - Rename columns
    - Change table properties
    - BUT YOU CANNOT change the PRIMARY KEY columns

    ```sql
    ALTER TABLE table1 ADD another_column text

    ALTER TABLE table1 DROP another_column;
    ```

  - SOURCE command allows you to execute a set of CQL statements from a file. The file name must be enclosed in single quotes. Example: `SOURCE './myscript.cql';`. Cqlsh will output the results of each command sequentially as it executes.

### Lesson 4: Partitions

To truly understand the data modeling, you must master partitioning concepts. Partitions give you an indication of where your data is in your data model as well as in the cluster. Determination of the partition key is a critical step. The partition key is always the first value in the primary key.

- Partitions
- Partition Keys
- Composite Partition Keys
- Clustering Columns

## Module 2:  Cassandra™ Building Blocks.

This module will cover the basics of Cassandra™ data model.

Upon completion of this module, you will be able to:

- Explore the concepts of denormalization
- And investigate collection, counters, and User Defined Types

### Lesson 1: De-normalization



## (31:29) Module 3: Data Modeling

## (1:04:22) Module 4: Practical Applications


## Video
 * [Apache Cassandra Database – Full Course for Beginners](https://www.youtube.com/watch?v=J-cSy5MeMOA)
	> [<img src="https://img.youtube.com/vi/J-cSy5MeMOA/0.jpg" width="200">](https://www.youtube.com/watch?v=J-cSy5MeMOA "Apache Cassandra Database – Full Course for Beginners by freeCodeCamp.org 250,408 views 1 hour, 8 minutes")
 * [Build scalable API-based Microservices with Spring Boot and Cassandra](https://www.youtube.com/watch?v=mL9oDZPJfwk)
	> [<img src="https://img.youtube.com/vi/mL9oDZPJfwk/0.jpg" width="200">](https://www.youtube.com/watch?v=mL9oDZPJfwk "Join us as we showcase how to build Scalable API Microservices using the power of Cassandra through an open-source data layer called Stargate! by DataStax Developers 15k views 1 hour, 21 minutes")
* [NoSQL Database Tutorial – Full Course for Beginners](https://www.youtube.com/watch?v=xh4gy1lbL2k)
	> [<img src="https://img.youtube.com/vi/xh4gy1lbL2k/0.jpg" width="200">](https://www.youtube.com/watch?v=xh4gy1lbL2k "NoSQL Database Tutorial – Full Course for Beginners by freeCodeCamp.org 821,555 views 2 hours, 54 minutes")
* [Build a Reactive app in Apache Cassandra™ with Spring Framework](https://www.youtube.com/watch?v=1aRbndIcXV4)
	> [<img src="https://img.youtube.com/vi/1aRbndIcXV4/0.jpg" width="200">](https://www.youtube.com/watch?v=1aRbndIcXV4 "It’s now possible to develop fully reactive Java applications with frameworks like Spring WebFlux and Reactor. In this interactive workshop we'll explore the power of these frameworks and get hands on with a reactive implementation of the popular Spring Pet Clinic reference application. by DataStax Developers 2.1K views 1 hours, 44 minutes")
 * [Advanced Data Modeling in Apache Cassandra](https://www.youtube.com/watch?v=u6pKIrfJgkU)
	> [<img src="https://img.youtube.com/vi/u6pKIrfJgkU/0.jpg" width="200">](https://www.youtube.com/watch?v=u6pKIrfJgkU "In this workshop we go beyond queries, partitions and tables to see how to design an efficient data model for highly-loaded applications. We will also cover some advanced use-cases and review some real-life examples with you! by DataStax Developers 15k views 1 hour, 21 minutes")
 * [Introduction to MachineLearning with Cassandra and Spark](https://www.youtube.com/watch?v=L5ishPnr1Y0)
	> [<img src="https://img.youtube.com/vi/L5ishPnr1Y0/0.jpg" width="200">](https://www.youtube.com/watch?v=L5ishPnr1Y0 "In this workshop we go beyond queries, partitions and tables to see how to design an efficient data model for highly-loaded applications. We will also cover some advanced use-cases and review some real-life examples with you! by DataStax Developers 15k views 1 hour, 21 minutes")

## References
1. [Introduction to Cassandra - Crash ](https://www.youtube.com/watch?v=YjYWsN1vek8&list=PL2g2h-wyI4SqCdxdiyi8enEyWvACcUa9R)
2. [K8ssandra, Apache Cassandra](https://www.youtube.com/playlist?list=PL2g2h-wyI4Sq_6MQEVn4fFmjSJ10IwVrU)
3. [Introdução ao Cassandra - Cassandra Day Brasil 2022](https://www.youtube.com/watch?v=iNOjDBpZ-CA&list=PL2g2h-wyI4SplwJPcSW86Nm75Zhj2b7B5)
4. [Cassandra Forward 2023](https://www.youtube.com/watch?v=mHfZP0gQq0k&list=PL2g2h-wyI4Spglm5OHOOmbHW68ZaE58Oo)
5. [Learn Apache Cassandra](https://www.youtube.com/watch?v=wOyQlbFM1Uk&list=PL2g2h-wyI4SqzmsHQZaYcK9uDmPuihPz2)
6. [Apache Cassandra 4.0](https://www.youtube.com/playlist?list=PL2g2h-wyI4Sor6qaGlbRv1htx0hmgmIs5)
7. [Cassandra interview questions answers](https://www.cloudduggu.com/cassandra/interview-questions-answers/)
