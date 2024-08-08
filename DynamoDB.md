# DynamoDB

Amazon DynamoDB is a fully managed NoSQL database service that provides fast and predictable performance with seamless scalability. As a key-value and document database, it is designed to handle large-scale, high-throughput applications. Here are some key features and characteristics of DynamoDB as a key-value database:

### Key Features

1. **Scalability**: DynamoDB automatically scales to accommodate your application's load. It can handle millions of requests per second and scale up and down based on demand without any manual intervention.

2. **Performance**: DynamoDB offers single-digit millisecond response times, making it suitable for applications requiring low latency.

3. **Fully Managed**: Being a fully managed service, DynamoDB handles tasks such as hardware provisioning, setup, configuration, replication, software patching, and cluster scaling.

4. **Key-Value Store**: In DynamoDB, data is stored as key-value pairs. Each item (record) is uniquely identified by a primary key, which can be a single attribute (partition key) or a combination of two attributes (partition key and sort key).

5. **Data Models**: In addition to the key-value model, DynamoDB supports document data models. This flexibility allows it to store and query complex data structures like JSON documents.

6. **Global Tables**: DynamoDB Global Tables provide a fully replicated, multi-region, multi-master database that ensures low-latency data access to users worldwide.

7. **Secondary Indexes**: DynamoDB supports both local secondary indexes (LSIs) and global secondary indexes (GSIs) to enable efficient querying on non-primary key attributes.

8. **Backup and Restore**: DynamoDB provides on-demand backup and restore capabilities, ensuring data protection and recovery.

9. **Streams**: DynamoDB Streams capture a time-ordered sequence of changes made to items in the table, which can be used for real-time processing or archiving.

10. **Security**: DynamoDB integrates with AWS Identity and Access Management (IAM) for fine-grained access control and supports encryption at rest and in transit.

### Usage Scenarios

- **Real-time applications**: Due to its low-latency responses, DynamoDB is suitable for gaming, IoT, and mobile applications.
- **E-commerce platforms**: Its scalability and high availability make it ideal for online retail applications.
- **Data lakes and analytics**: DynamoDB can be used to store and process large volumes of data for analytical applications.
- **Content management**: Its flexible data model supports the storage of various content types, making it useful for content management systems.

### Example of Key-Value Data

```json
{
  "CustomerId": "12345",
  "Name": "John Doe",
  "Email": "johndoe@example.com",
  "Address": {
    "Street": "123 Elm St",
    "City": "Anytown",
    "State": "CA",
    "ZipCode": "90210"
  }
}
```

In this example, "CustomerId" is the primary key, and the rest of the attributes represent the value associated with this key. DynamoDB allows you to perform operations such as `GetItem`, `PutItem`, `UpdateItem`, and `DeleteItem` on these key-value pairs.

DynamoDB's key-value nature, combined with its powerful features and seamless integration with other AWS services, makes it a versatile and robust choice for a wide range of applications.

## **Summary of the Paper: "Amazon DynamoDB: A Scalable, Predictably Performant, and Fully Managed NoSQL Database Service"** [[1]](./img/x/sahnlam/atc22-elhemali.pdf)

**Abstract:**
Amazon DynamoDB is a NoSQL cloud database service known for its consistent performance at any scale. The paper highlights DynamoDB's ability to handle large-scale operations, such as the 66-hour Amazon Prime Day shopping event in 2021, where it processed trillions of API calls with high availability and low latency. The service has evolved since its 2012 launch to address issues like fairness, traffic imbalance, and automated operations without compromising performance or availability.

**Introduction:**
DynamoDB is a foundational AWS service that supports high-traffic applications like Alexa and Amazon.com. It is designed for consistent low-latency performance at any scale, which is crucial for applications that depend on it. The service integrates six fundamental properties: fully managed, multi-tenant architecture, boundless scalability, predictable performance, high availability, and flexible use cases.

**Evolution and Challenges:**
The paper details DynamoDB's evolution from its predecessor Dynamo, which was a single-tenant system with operational complexities. DynamoDB combines the best aspects of Dynamo's scalability and performance with SimpleDB's ease of administration. The service's multi-tenant architecture ensures high resource utilization and cost savings for customers. DynamoDB also offers elastic table growth and predictable low-latency performance, making it suitable for diverse applications.

**Key Lessons Learned:**
1. Adapting to traffic patterns improves customer experience.
2. Continuous verification of data-at-rest enhances durability.
3. Maintaining high availability requires careful operational discipline and tooling.
4. Designing for predictability over absolute efficiency improves system stability.

**Architecture Overview:**
DynamoDB tables consist of items identified by a primary key. The architecture supports CRUD operations (Create, Read, Update, Delete) through a simple API. The system uses a distributed data placement and request routing algorithm to maintain low-latency performance as tables grow. DynamoDB replicates data across multiple Availability Zones for high availability and durability, with optional geo-replication for global tables.

**Conclusion:**
The paper concludes by emphasizing DynamoDB's ability to provide a single-tenant experience in a multi-tenant architecture, meeting the needs of a wide range of customer workloads. The service's ongoing evolution focuses on maintaining its core properties of durability, availability, scalability, and predictable performance.

**Paper Structure:**
- Section 2: History and origins from Dynamo
- Section 3: Architectural overview
- Section 4: Journey from provisioned to on-demand tables
- Section 5: Ensuring strong durability
- Section 6: Handling availability challenges
- Section 7: Experimental results (Yahoo! Cloud Serving Benchmark)
- Section 8: Conclusion

DynamoDB's design and operational strategies ensure it remains a robust, reliable, and high-performing NoSQL database service, capable of handling the most demanding applications and workloads.

![DynamoDB](./img/x/sahnlam/DynamoDB%20architecture.jpg)

## Surprising to me: The storage node is B-tree based, not LSM tree - like I assumed for large scale NoSQL databases.

Yes, it's indeed interesting. While many NoSQL databases like Apache Cassandra and Google Bigtable use LSM (Log-Structured Merge) trees to handle large-scale write-intensive workloads efficiently, DynamoDB's choice of a B-tree-based storage node is notable. B-trees are typically favored for their performance in read-heavy workloads and their ability to maintain sorted data, which can be beneficial for range queries and ordered scans.

This design choice in DynamoDB likely reflects a balance between write performance, read efficiency, and the specific needs of DynamoDB's use cases. By leveraging B-trees, DynamoDB ensures consistent read performance and efficient data access patterns, which are crucial for the wide range of applications it supports.

![LSM vs B-tree](./img/x/rocky%20bhatia/LSM%20vs%20B-Tree%20-%20The%20backbone%20of%20Modern%20Databases.gif)

## Solution overview

The reference solution uses a container-based Python application with a DynamoDB table as the data store. Along with Amazon ECS, the solution uses a combination of AWS services for continuous integration and continuous delivery (CI/CD), Amazon Elastic Container Registry (Amazon ECR) for container registry, AWS AppConfig to host application configuration, and Amazon CloudWatch Logs to manage logs. AWS Virtual Private Cloud (VPC) manages the network setup and Amazon Application Load Balancer (ALB) routes external traffic to the Amazon ECS service. The factors described in the next section provide details on how the different parts of the reference solution achieve the best practices recommended by the methodology. The high-level architecture of the reference solution is shown below. 

![Solution overview](./img/Twelve-Factor/Blog-Architecture-Page-1.png)

### The twelve factors of the reference solution
- Backing services – Your application must treat any service that it uses to perform its business function as an attached resource. This may be an RDS database, a DynamoDB table, an S3 bucket, or even a non-AWS service. The attached resource semantics prescribes that these resources are made available to the application as a configuration. Your application will be aware of the attached resource, but not be tightly coupled to it. For a DynamoDB table, it is sufficient to be aware of the table name, which is the resource identifier. In the reference solution, the name of the DynamoDB table comes from AppConfig, allowing you to switch out one table for another without any application downtime. This can be useful in scenarios where you are experimenting with different data sets to achieve A/B testing.

## Compare to Azure CosmosDB, GCP BigTable and Redis, ScyllaDB and Ignite

Comparing Amazon DynamoDB with Azure Cosmos DB, Google Cloud Bigtable, Redis, ScyllaDB, and Apache Ignite can help you understand their strengths and use cases. Here's a comparison across several key dimensions:

### Amazon DynamoDB

- **Type**: Fully managed NoSQL key-value and document database
- **Scalability**: Automatically scales, supports on-demand and provisioned capacity modes
- **Performance**: Single-digit millisecond latency, suitable for high-throughput applications
- **Data Model**: Key-value and document
- **Consistency**: Offers eventual and strong consistency models
- **Global Distribution**: Supports multi-region, multi-master with Global Tables
- **Secondary Indexes**: Supports LSIs and GSIs
- **Security**: Integrated with AWS IAM, encryption at rest and in transit
- **Use Cases**: Real-time applications, e-commerce, content management, data lakes

### Azure Cosmos DB

- **Type**: Fully managed multi-model database service (key-value, document, graph, column-family)
- **Scalability**: Automatic and manual scaling, global distribution with multi-region writes
- **Performance**: Low latency, guaranteed by SLAs for throughput, latency, availability, and consistency
- **Data Model**: Key-value, document, graph, column-family
- **Consistency**: Five consistency levels (strong, bounded staleness, session, consistent prefix, eventual)
- **Global Distribution**: Native support for global distribution, multi-master replication
- **Secondary Indexes**: Automatic indexing, customizable indexing policies
- **Security**: Integrated with Azure Active Directory, encryption at rest and in transit
- **Use Cases**: IoT, retail, gaming, web and mobile applications

### Google Cloud Bigtable

- **Type**: Fully managed wide-column NoSQL database
- **Scalability**: Scales horizontally to handle petabytes of data and millions of operations per second
- **Performance**: Low latency, high throughput, optimized for large analytical and operational workloads
- **Data Model**: Wide-column (similar to HBase)
- **Consistency**: Strong consistency within a single region, eventual consistency across multiple regions
- **Global Distribution**: Multi-region replication is supported but more complex than DynamoDB or Cosmos DB
- **Secondary Indexes**: Does not support secondary indexes natively
- **Security**: Integrated with Google Cloud IAM, encryption at rest and in transit
- **Use Cases**: Time-series data, IoT, financial data analysis, large-scale analytical applications

### Redis

- **Type**: In-memory key-value store, supports various data structures (strings, hashes, lists, sets, sorted sets)
- **Scalability**: Scales horizontally using clustering and sharding
- **Performance**: Extremely low latency due to in-memory data storage
- **Data Model**: Key-value, with support for complex data structures
- **Consistency**: Eventual consistency with asynchronous replication
- **Global Distribution**: Supports clustering across regions
- **Secondary Indexes**: Does not support secondary indexes natively
- **Security**: TLS encryption, authentication, integration with cloud IAM services
- **Use Cases**: Caching, session management, real-time analytics, message brokering

### ScyllaDB

- **Type**: NoSQL database compatible with Apache Cassandra
- **Scalability**: Horizontally scalable, designed to handle petabytes of data with high throughput
- **Performance**: High performance with low latency, optimized for modern hardware
- **Data Model**: Wide-column store, similar to Cassandra
- **Consistency**: Tunable consistency, from strong to eventual
- **Global Distribution**: Supports multi-region replication
- **Secondary Indexes**: Supports secondary indexes
- **Security**: TLS encryption, authentication, role-based access control
- **Use Cases**: Time-series data, IoT, large-scale data ingestion, real-time analytics

### Apache Ignite

- **Type**: In-memory computing platform, supports key-value store, SQL, and compute grid
- **Scalability**: Horizontally scalable, supports both in-memory and on-disk storage
- **Performance**: Extremely low latency due to in-memory data processing
- **Data Model**: Key-value, SQL, and compute grid
- **Consistency**: Supports both strong and eventual consistency
- **Global Distribution**: Can be configured for multi-region deployments
- **Secondary Indexes**: Supports secondary indexes
- **Security**: SSL/TLS encryption, authentication, role-based access control
- **Use Cases**: Real-time analytics, high-performance computing, distributed caching, transaction processing

### Summary

- **DynamoDB**: Excellent for high-throughput, low-latency applications with automatic scaling and global distribution.
- **Azure Cosmos DB**: Versatile multi-model database with strong consistency and global distribution options.
- **Google Cloud Bigtable**: Best for large-scale analytical and operational workloads, especially with time-series data.
- **Redis**: Ideal for low-latency in-memory operations, caching, and real-time analytics.
- **ScyllaDB**: High-performance NoSQL database compatible with Cassandra, suitable for large-scale real-time applications.
- **Apache Ignite**: Comprehensive in-memory computing platform, excellent for real-time analytics and high-performance computing.

Each database has its unique strengths, making them suitable for different types of applications and workloads.




## Videos

 * [Week 5 DynamoDB Stream](https://www.youtube.com/watch?v=zGnzM_YdMJU)
	> [<img src="https://img.youtube.com/vi/zGnzM_YdMJU/0.jpg" width="200">](https://www.youtube.com/watch?v=zGnzM_YdMJU "Week 5 DynamoDB Stream by ExamPro 1,371 views 43 minutes")

 * [Serverless: Do Zero ao Deploy com API Gateway, Lambda e DynamoDB](https://www.youtube.com/watch?v=kpjkSLXHN5E)
	> [<img src="https://img.youtube.com/vi/kpjkSLXHN5E/0.jpg" width="200">](https://www.youtube.com/watch?v=kpjkSLXHN5E "Serverless: Do Zero ao Deploy com API Gateway, Lambda e DynamoDB by Full Cycle 8,937 views 1 hour")

## References
1. https://aws.amazon.com/pt/dynamodb/getting-started/
2. [Amazon DynamoDB: Building NoSQL Database-Driven Applications](https://www.edx.org/learn/amazon-web-services-aws/amazon-web-services-amazon-dynamodb-building-nosql-database-driven-applications)
3. [AWS Tutorials - DynamoDB and Database Migration Service](https://www.udemy.com/course/namrata-h-shah-aws-tutorials-dynamodb-and-database-migration-service/)
4. 