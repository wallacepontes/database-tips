# CDC

Change Data Capture (CDC) is a technology used in database management that captures and records changes made to data in a source database and replicates these changes to a target system in real-time or near real-time. CDC is particularly valuable for scenarios where it's important to keep multiple systems synchronized with the most up-to-date data. Here are some key aspects and reasons for using CDC:

![CDC](./img/x/sahnlam/How%20does%20CDC%20work%20-%20Change%20Data%20Capture.jpg)

**1. Real-Time Analytics:** CDC enables organizations to have access to real-time data updates. This is crucial for making informed decisions based on the most current information available.

**2. Resource Efficiency:** Instead of continuously querying the entire source database for changes, CDC minimizes the computational and I/O overhead on the source system by only capturing and propagating the specific changes (INSERTs, UPDATEs, DELETEs) that occur.

**3. Data Synchronization:** CDC ensures that all systems connected to the source database reflect the latest state of the data. This is essential for maintaining data consistency across various platforms and applications.

**4. System Recovery:** CDC can be used to reconstruct the state of a system by replaying the sequence of changes captured. This is valuable for recovering from failures or restoring databases to a specific point in time.

**Types of CDC:**

1. **Trigger-Based CDC:** This method relies on database triggers, which are database objects that automatically execute code (such as capturing change information) when certain events (e.g., INSERT, UPDATE, DELETE) occur on a table.

2. **Log-Based CDC:** Log-based CDC directly reads and interprets changes from the transaction logs of the source database. This method is considered highly efficient and is often used for real-time CDC.

3. **Timestamp-Based CDC:** Timestamp-based CDC relies on timestamp columns in tables to identify newly modified records. This approach is typically used when more granular tracking of changes is required.

**Challenges with CDC:**

1. **Data Integrity:** Ensuring that all changes are accurately captured and replicated is critical to maintaining data reliability and consistency.

2. **Scalability:** As data volumes increase, CDC systems must be designed to handle larger workloads and scale effectively.

3. **Latency:** Minimizing the time it takes for changes to propagate from the source to the target systems is crucial, especially for applications that require real-time data.

**Tools and Technologies for CDC:**

1. **Kafka:** Kafka is a distributed streaming platform that is well-suited for managing the flow of change events in real-time. It is commonly used in conjunction with CDC systems.

2. **Debezium:** Debezium is an open-source CDC software that integrates with Kafka and is designed to capture and stream changes from various databases, including relational databases like MySQL, PostgreSQL, and more.

In modern data architectures, CDC has become a fundamental component due to its ability to provide real-time data, maintain data consistency, and support system recovery. It plays a crucial role in enabling organizations to stay agile and make data-driven decisions based on the most current information available.

## What is Change Data Capture (CDC)?

What is Change Data Capture (CDC)?

In database terms, CDC captures changes made in the data source, such as INSERT, UPDATE, and DELETE operations, and replicates them into a target system, typically in real-time or near real-time.

👉 Example: Imagine a Users table in an RDBMS where new rows are inserted whenever new users sign up, and existing rows are updated when users change their profile information. With CDC, instead of querying the entire Users table frequently to check for changes, you would automatically be notified of just the new and updated rows. This data can be ingested into an analytics system for further processing without affecting the source database's performance.

Why Do We Need CDC?
-----------------------
Real-Time Analytics: As the need for immediate insights grows, CDC's ability to offer real-time data becomes invaluable.

Resource Efficiency: CDC minimizes computational and I/O overhead on your source database by only capturing changes.

Data Synchronization: Ensures all your systems reflect the latest state of your data, maintaining data accuracy across platforms.

System Recovery: Enables effective state reconstruction by using a sequence of captured changes.

Types of CDC
-----------------
Trigger-Based CDC: Employs database triggers to capture changes.

Log-Based CDC: Directly reads changes from the database transaction logs.

Timestamp-Based CDC: Relies on timestamp columns in tables to identify newly modified records.

Challenges with CDC
--------------------
Data Integrity: Capturing all changes accurately is crucial for maintaining data reliability.

Scalability: As data volumes increase, the CDC system must adapt and scale effectively.

Latency: Minimizing the time it takes for data changes to propagate to target systems is a priority, especially in real-time applications.

Tools and Technologies
----------------------
Kafka: A distributed streaming platform, ideal for managing the flow of change events.

Debezium: An open-source CDC software that integrates with Kafka, designed to capture and stream changes from various databases.

CDC has proven to be more than just a "good-to-have" feature; it's becoming increasingly essential for modern data infrastructures.

Its ability to offer real-time data, ensure data consistency, and aid in system recovery makes it a cornerstone in today's data strategies.

## CDC vs Outbox Pattern

Choosing between the Outbox pattern and Change Data Capture (CDC) depends on the specific needs and context of your application. Here's a comparison of the two approaches:

### Outbox Pattern

**Pros:**
1. **Transactional Consistency**: Ensures that the changes are only published if the transaction succeeds. This guarantees that the state changes in your database and the messages sent to the message broker are consistent.
2. **Simplicity**: Often simpler to implement in applications that already use a relational database and message broker.
3. **Explicit Control**: Developers have explicit control over what gets written to the outbox table and when it's published.

**Cons:**
1. **Increased Complexity**: Requires modifications to the application code to write to the outbox table and to a separate process to read from the outbox table and publish messages.
2. **Performance Overhead**: Writing to an additional table within the transaction can introduce some performance overhead.
3. **Maintenance**: The outbox table needs to be managed and cleaned up periodically to prevent it from growing indefinitely.

### Change Data Capture (CDC)

**Pros:**
1. **No Application Changes**: Works by monitoring the database's transaction log, so no changes to application code are necessary.
2. **Real-Time Updates**: Provides near real-time data changes by capturing changes as they happen in the database.
3. **Broad Database Support**: Many modern databases support CDC, either natively or through third-party tools like Debezium.

**Cons:**
1. **Complexity in Setup**: Setting up and managing CDC can be complex, especially if the database doesn't have native CDC support.
2. **Eventual Consistency**: Depending on the implementation, there can be a slight delay between the database change and the event being processed, leading to eventual consistency rather than strong consistency.
3. **Performance Impact**: Depending on the volume of changes and the efficiency of the CDC implementation, there can be a performance impact on the database.

### Best Choice

The best choice depends on your specific requirements:

- **Use the Outbox Pattern if**:
  - You need strong transactional consistency between the database and message broker.
  - Your application can tolerate the additional complexity and performance overhead.
  - You prefer explicit control over the message publishing process.

- **Use CDC if**:
  - You want to minimize changes to your application code.
  - You can tolerate eventual consistency.
  - Your database supports efficient CDC, and you are comfortable setting it up and managing it.

In summary, the Outbox pattern is ideal for applications that require strong consistency and can handle the additional complexity, while CDC is suitable for applications that prioritize minimal application changes and can work with eventual consistency.

## References
1. https://medium.com/@gabrielqueiroz/como-usar-o-kafka-jdbc-sink-connector-391ded610f3
2. https://docs.confluent.io/platform/current/platform-quickstart.html#quickstart
3. 