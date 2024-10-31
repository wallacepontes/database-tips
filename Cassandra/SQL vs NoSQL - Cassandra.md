# SQL vs NoSQL

## First define sql and nosql database 

SQL (Structured Query Language) databases are relational databases that use structured query language for defining and manipulating data. They organize data into tables with rows and columns, and they enforce a predefined schema for the data. Examples include MySQL, PostgreSQL, and Oracle.

NoSQL (Not Only SQL) databases are non-relational databases that provide flexible schema designs for storing, retrieving, and managing data. They can handle large volumes of structured, semi-structured, and unstructured data. Examples include MongoDB, Cassandra, and Redis.

## Which user cases have to each of them?

SQL databases are well-suited for applications requiring complex queries, transactions, and strict consistency, such as:

1. **Transactional Applications**: Applications like banking systems, e-commerce platforms, and accounting software where data integrity and ACID (Atomicity, Consistency, Isolation, Durability) compliance are crucial.

2. **Relational Data**: When the data has a clear structure and relationships between entities that need to be maintained, such as in CRM (Customer Relationship Management) systems.

3. **Complex Queries**: Applications that require complex joins, aggregations, and multi-table transactions benefit from the powerful query capabilities of SQL databases.

NoSQL databases, on the other hand, are suitable for scenarios where flexibility, scalability, and performance are more important than rigid data consistency, such as:

1. **Big Data and Analytics**: NoSQL databases excel at handling large volumes of data with varying structures, making them ideal for big data analytics and real-time analytics platforms.

2. **Highly Scalable Systems**: Applications that need to scale horizontally across multiple servers or nodes, like social networks, content management systems, and IoT (Internet of Things) platforms, can benefit from the distributed architecture of NoSQL databases.

3. **Fast Data Retrieval**: When data needs to be retrieved quickly without complex joins or transactions, NoSQL databases can provide faster read and write operations, making them suitable for real-time applications and caching layers.

## Is it possible image a system using both database using CQRS pattern ?

Yes, it's possible to design a system that uses both SQL and NoSQL databases following the CQRS (Command Query Responsibility Segregation) pattern. 

In a CQRS architecture, the system is divided into two separate parts: the Command side and the Query side. The Command side handles operations that modify the application's state (e.g., creating, updating, or deleting data), while the Query side handles operations that retrieve data (e.g., reading data for display purposes).

Here's how you could incorporate both types of databases in a CQRS-based system:

1. **Command Side (Write Model)**:
   - Use a SQL database for handling transactional operations and maintaining data consistency. SQL databases are well-suited for enforcing ACID properties and managing complex transactions.
   - This side of the system handles commands that mutate the data state, such as creating new records, updating existing ones, or deleting data.

2. **Query Side (Read Model)**:
   - Use a NoSQL database for handling read-heavy operations and supporting flexible data structures. NoSQL databases can handle large volumes of read operations efficiently and can accommodate varying data structures.
   - This side of the system handles queries for retrieving data, which may involve denormalized or pre-computed views optimized for read performance.

By separating the write and read responsibilities, you can tailor each part of the system to the specific requirements of its workload. This approach allows for better scalability, performance, and flexibility in handling different types of data operations.

## SQL vs. NoSQL Explained 
   > I don't know but as a beginner I have noticed many of these videos LACK the most important sauce : ACTUAL USE CASES! If you would have featured examples like let's say you're building an app that does this or that for you might want to use SQL for this part of the scope BECAUSE etc... and for that part of the scope NoSQL might be preferred because etc.... There you go, I have just shared a framework for your next video lol. Thank you anyway for the content it shows the genuine interest to educate but could be better especially if you want to educate beginners.

That's a great point! Understanding the practical applications of SQL and NoSQL databases can really help beginners grasp their differences and when to use each one. Let's break it down with some examples:

1. **E-commerce Application:**
   - SQL: If you're building an e-commerce platform where you need to manage inventory, orders, and customer data with complex relationships (e.g., many-to-many relationships between products and orders), SQL databases like MySQL or PostgreSQL would be a good choice. You can ensure data integrity through transactions and ACID properties.
   - NoSQL: On the other hand, if you're focusing on handling a massive amount of user-generated data, such as user profiles, product reviews, or session data, NoSQL databases like MongoDB or Cassandra might be preferred. NoSQL databases are great for scalability and flexibility, allowing you to store unstructured or semi-structured data efficiently.

2. **Content Management System (CMS):**
   - SQL: For a CMS where you need to organize content hierarchically, manage user permissions, and maintain a consistent schema for articles, pages, and users, SQL databases are a good fit. You can enforce relationships between different content types and ensure data consistency.
   - NoSQL: If your CMS prioritizes flexibility and fast iteration, especially when dealing with diverse content types (e.g., blog posts, videos, images) with varying attributes, a NoSQL database might be more suitable. NoSQL databases allow you to store different types of content without a fixed schema, enabling quick modifications and scaling.

3. **Real-time Analytics Platform:**
   - SQL: When building a real-time analytics platform where you need to perform complex queries on structured data, such as aggregating metrics, generating reports, and running ad-hoc queries, SQL databases excel. With SQL, you can leverage powerful query optimization techniques and indexing to ensure fast and efficient data retrieval.
   - NoSQL: If your analytics platform focuses on handling large volumes of semi-structured or unstructured data streams, such as clickstream data, social media feeds, or sensor data, a NoSQL database like Apache Kafka or Elasticsearch might be preferred. NoSQL databases can handle high throughput and offer horizontal scalability, making them suitable for real-time data ingestion and analysis.

By understanding these use cases, beginners can make more informed decisions when choosing between SQL and NoSQL databases for their projects. Thanks for providing a helpful framework!

## Courses Data Engineering
Learn SQL for Free in 2024: Top 15 Resources

 1. SQLBolt: https://sqlbolt.com/
 
 2. Khan Academy SQL Course: https://www.khanacademy.org/computing/computer-programming/sql
 
 3. Codecademy Learn SQL: https://lnkd.in/efMKFkfX
 
 4. W3Schools SQL Tutorial: https://lnkd.in/eYqMX692
 
 5. MySQL Tutorial: https://lnkd.in/ekgiHxBw
 
 6. PostgreSQL Tutorial: https://lnkd.in/eYhaurMN
 
 7. SQLZoo: https://sqlzoo.net/
 
 8. HackerRank SQL Challenges: https://lnkd.in/eUU2hGzd
 
 9. LeetCode SQL Problems: https://lnkd.in/eTA2Gtp8
 
10. DataCamp Intro to SQL: https://lnkd.in/evn_VTKR
 
11. Udacity: https://lnkd.in/eAiMs3zs
 
12. Coursera SQL for Data Science: https://lnkd.in/e5-U8sTe
 
13. edX Introduction to SQL: https://lnkd.in/eRVhFASa
 
14. MIT : https://lnkd.in/eT3Xv6vr
 
15. Udemy : https://lnkd.in/erigbcNs
16. 
Learn One RDBMS and one NoSQL database.

1. Learn Data Warehousing concepts - https://lnkd.in/eKnVbFAB
2. Learn MySql - https://lnkd.in/efk-Mi3c
3. Learn and practice SQL - https://lnkd.in/efMKFkfX
4. Learn Azure Cosmos DB - https://lnkd.in/eNVyc6Mq

🔶 Learn Python:

 https://lnkd.in/e5rCbvP8

🔶 Learn Database and SQL :

1. SQL - https://lnkd.in/efMKFkfX
 
2. Learn MySql - https://lnkd.in/efk-Mi3c
 
3. Learn DynamoDB - https://lnkd.in/eJUS6JS4
 
 
🔶 Learn PySpark :

https://lnkd.in/exwA2hKz


🔶 Learn Bash, Airflow and Kafka:

https://lnkd.in/eyN6u2yd

🔶 Learn Git -

https://lnkd.in/eX_Q8s99

🔶 Learn Basics of CICD -

https://lnkd.in/epKGivFY

🔶 Learn Data Warehousing 

https://lnkd.in/eKnVbFAB

🔶 Learn DBT:

 https://lnkd.in/eG9eaEuE

👉 Create a standout resume by showcasing your skills and experience in data engineering.

👉 Enhance your portfolio by completing various data engineering projects and sharing them on Git.

👉 Seek out opportunities in data engineering by applying to relevant positions


Optional Learning - 

1. Learn Data Structure and Algorithms

2. Learn Scala

3. Learn AWS or Azure services dedicated for data engineering work.

4. Learn DataBricks or Snowflake

5. Learn Data Lakes

## Video

 * [Apache Cassandra (NoSQL de Gente Grande e que Paga Muito Bem) // Dicionário do Programador](https://www.youtube.com/watch?v=4pgdLCu-8zs)
	> [<img src="https://img.youtube.com/vi/4pgdLCu-8zs/0.jpg" width="200">](https://www.youtube.com/watch?v=4pgdLCu-8zs "Apache Cassandra (NoSQL de Gente Grande e que Paga Muito Bem) // Dicionário do Programador by Código Fonte TV 18,407 views 16 minutes")

 * [SQL vs. NoSQL Explained (in 4 Minutes)](https://www.youtube.com/watch?v=_Ss42Vb1SU4)
	> [<img src="https://img.youtube.com/vi/_Ss42Vb1SU4/0.jpg" width="200">](https://www.youtube.com/watch?v=_Ss42Vb1SU4 "I don't know but as a beginner I have noticed many of these videos LACK the most important sauce : ACTUAL USE CASES! If you would have featured examples like let's say you're building an app that does this or that for you might want to use SQL for this part of the scope BECAUSE etc... and for that part of the scope NoSQL might be preferred because etc.... There you go, I have just shared a framework for your next video lol. Thank you anyway for the content it shows the genuine interest to educate but could be better especially if you want to educate beginners. by Exponent 130K views 4 minutes")