# Different types of unique identifier systems

## UUID vs ULID vs TLID

UUID, ULID, and TLID are different types of unique identifier systems used in various applications, particularly in databases and distributed systems. Here's a detailed comparison of these identifiers:

### UUID (Universally Unique Identifier)

#### Overview
- **Format:** UUIDs are 128-bit values usually represented as 32 hexadecimal characters, divided into five groups separated by hyphens, in the form `8-4-4-4-12`.
- **Example:** `550e8400-e29b-41d4-a716-446655440000`

#### Versions
- **UUIDv1:** Timestamp and MAC address-based.
- **UUIDv4:** Randomly generated.
- **UUIDv5:** Name-based, using SHA-1 hashing.

#### Characteristics
- **Collision Probability:** Extremely low.
- **Ordering:** Not time-ordered (except for UUIDv1).
- **Length:** 128 bits.
- **Use Cases:** General-purpose unique identifier, used in databases, distributed systems, etc.

### ULID (Universally Unique Lexicographically Sortable Identifier)

#### Overview
- **Format:** ULIDs are 128-bit values represented as 26 alphanumeric characters.
- **Example:** `01ARZ3NDEKTSV4RRFFQ69G5FAV`

#### Characteristics
- **Collision Probability:** Extremely low.
- **Ordering:** Lexicographically sortable, first 48 bits represent a timestamp.
- **Length:** 128 bits.
- **Use Cases:** Systems requiring sortable unique identifiers, log streams, databases.

#### Advantages
- **Sortability:** Can be sorted lexicographically, making it suitable for ordered data storage and retrieval.
- **Readability:** More compact and easier to read compared to UUIDs.

### TLID (Timestamped Lexicographically Sortable Identifier)

#### Overview
- **Format:** TLIDs are typically designed to be sortable and include timestamp information.
- **Characteristics:** Implementation specifics can vary as TLID is a more general concept.
- **Use Cases:** Systems needing identifiers that encode creation time and can be sorted.

#### Characteristics
- **Collision Probability:** Depends on the implementation; typically low.
- **Ordering:** Lexicographically sortable, includes timestamp.
- **Length:** Variable, but generally similar to ULIDs.
- **Use Cases:** Distributed systems, databases, systems requiring time-based sorting.

### Comparison Summary

| Characteristic    | UUID                      | ULID                        | TLID                      |
|-------------------|---------------------------|-----------------------------|---------------------------|
| **Format**        | 128-bit (hexadecimal)     | 128-bit (alphanumeric)      | Variable, timestamp-based |
| **Example**       | `550e8400-e29b-41d4-a716-446655440000` | `01ARZ3NDEKTSV4RRFFQ69G5FAV` | Implementation-dependent  |
| **Collision Probability** | Extremely low      | Extremely low               | Low                       |
| **Ordering**      | Not time-ordered (except UUIDv1) | Lexicographically sortable | Lexicographically sortable|
| **Length**        | 128 bits                  | 128 bits                    | Variable                  |
| **Use Cases**     | General-purpose, databases, distributed systems | Ordered storage, log streams, databases | Time-based sorting, distributed systems |

### Choosing Between Them

- **UUID** is suitable for general-purpose unique identification where sortability and readability are not primary concerns.
- **ULID** is ideal for scenarios where lexicographical order is important, such as ordered log streams or databases that benefit from time-ordered records.
- **TLID** is useful when identifiers need to be sortable and include a timestamp, but the specifics can vary depending on the implementation and requirements.

Each identifier type has its strengths and is best suited for different use cases. The choice depends on the specific needs of the application, such as the necessity for sortability, readability, and collision resistance.

## The best UUID type for a database Primary Key

### 𝗦𝗵𝗼𝘂𝗹𝗱 𝘆𝗼𝘂 𝘀𝘁𝗼𝗽 𝘂𝘀𝗶𝗻𝗴 𝗨𝗨𝗜𝗗𝘀? 
 
UUIDs (Guid in C#) are widely used as unique identifiers in databases. UUIDs are random, which makes them popular in distributed systems. 
 
However, UUIDs have some drawbacks: 
 
1. UUIDs slow down database inserts. Each insert must update the clustered index, a B+ tree. Because UUIDs are random, this is an expensive operation as it requires rebalancing the tree 
 
2. Higher storage costs. A UUID is 128 bits long, and it's even longer if you store it in human-readable format as a string. 
 
So, let me introduce you to ULIDs. 
 
ULID attempts to solve the drawbacks of UUID. It's also 128-bit, so it's compatible with a UUID. However, unlike a UUID, ULIDs are sortable. The first 40 bits of a ULID represent a timestamp, making ULIDs monotonically increasing. 
 
There's a .NET package that implements the ULID spec, so you can start using it immediately. However, you'll need to write some code if you want ULID to work with popular ORMs. 
 
What do you think about ULIDs? 

![UUID vs ULID](./img/x/milan%20jovanovic/UUIDs%20vs%20ULID.jpg)

## ULIDs offer some advantages over UUIDs

ULIDs (Universally Unique Lexicographically Sortable Identifiers) offer some advantages over UUIDs (Universally Unique Identifiers), addressing specific issues related to the performance and storage costs associated with UUIDs in database systems. Here's a detailed comparison and analysis:

### Drawbacks of UUIDs

1. **Database Insert Performance**:
   - **Issue**: UUIDs are random, leading to non-sequential inserts in the database's clustered index (typically a B+ tree). This randomness causes frequent rebalancing and fragmentation, which can significantly slow down insert operations.
   - **Impact**: As the database size grows, the performance degradation becomes more noticeable, leading to slower write times and increased maintenance.

2. **Storage Costs**:
   - **Issue**: UUIDs are 128 bits long. When stored in a human-readable format (e.g., as a string), they take up even more space.
   - **Impact**: This increases storage requirements and can affect indexing performance due to the larger size of the identifiers.

### Benefits of ULIDs

1. **Sortability and Insert Performance**:
   - **Advantage**: The first 40 bits of a ULID represent a timestamp, making ULIDs lexicographically sortable. This means new ULIDs are typically inserted at the end of the index, reducing the need for rebalancing and fragmentation.
   - **Impact**: This can significantly improve insert performance, especially in systems with high write throughput.

2. **Compatibility and Storage**:
   - **Advantage**: ULIDs are also 128 bits long, maintaining compatibility with existing systems that use UUIDs.
   - **Impact**: Storage requirements remain the same as UUIDs when using the binary representation, but the benefit comes from the improved performance and potentially reduced fragmentation in the indexes.

### Implementation Considerations

- **.NET Support**: There is a .NET package for ULID, which simplifies integration into .NET applications. However, some work might be needed to ensure compatibility with ORMs (Object-Relational Mappers) like Entity Framework or Dapper.
- **Code Changes**: Depending on your current system's reliance on UUIDs, some code changes will be necessary to replace UUID generation with ULID generation. This includes ensuring that all parts of the application and database schema that interact with IDs are updated accordingly.

### Practical Considerations

- **Compatibility**: Ensure that your system can handle the transition from UUIDs to ULIDs without disrupting existing functionality.
- **Performance Testing**: Before fully committing to ULIDs, conduct performance tests in your specific environment to measure the actual benefits.
- **Index Strategy**: Review your indexing strategy to take full advantage of ULIDs' sortable nature.

### Conclusion

Switching from UUIDs to ULIDs can provide tangible benefits, particularly in improving database insert performance and managing storage more efficiently. However, the decision should be based on specific use case requirements, potential performance gains, and the effort required for implementation. Conduct thorough testing and planning to ensure a smooth transition and to fully realize the advantages of using ULIDs.

## 【[uuid-database-primary-key Vlad Mihalcea](https://vladmihalcea.com/uuid-database-primary-key/)】

UUIDs (Universally Unique Identifiers) are commonly used as unique keys in databases, particularly in distributed systems due to their random nature. However, they have some significant drawbacks:

1. **Performance Impact on Database Inserts**: UUIDs can slow down database inserts. This is because each insert operation requires updating the clustered index, which is a B+ tree. Due to the random nature of UUIDs, these updates necessitate frequent rebalancing of the tree, making the operation costly【[Vlad Mihalcea](https://vladmihalcea.com/books/high-performance-java-persistence/)】.

2. **Higher Storage Costs**: UUIDs are 128 bits long. When stored in a human-readable format as strings, they consume even more space. This can lead to higher storage requirements and potentially higher costs.

To address these issues, **ULIDs (Universally Unique Lexicographically Sortable Identifiers)** are proposed as an alternative. ULIDs offer several advantages:

- **Sortable**: Unlike UUIDs, ULIDs are sortable. The first 48 bits of a ULID represent a timestamp, making them monotonically increasing. This property can significantly improve performance for database operations that rely on ordered data.
- **Compatibility**: ULIDs maintain the same 128-bit length as UUIDs, ensuring compatibility while offering the benefits of sortability.

There are .NET packages available that implement the ULID specification, enabling immediate use. However, integrating ULIDs with popular Object-Relational Mappers (ORMs) may require some additional coding effort.

Overall, ULIDs can be a superior choice over UUIDs, especially for applications where the sortability of identifiers and performance optimization of database operations are critical. For further insights and practical implementations, Vlad Mihalcea's detailed discussions and performance analyses provide valuable guidance.


## is TSID a much better idea?

**TSIDs (Time Sortable Identifiers)** are another alternative to UUIDs and ULIDs that aim to address similar issues with database performance and storage efficiency. Let's delve into the benefits and considerations of using TSIDs compared to UUIDs and ULIDs.

### Advantages of TSIDs

1. **Time-based Component**:
   - **Sortability**: Like ULIDs, TSIDs include a time-based component, which allows them to be sorted chronologically. This can improve database performance for insert operations and queries that benefit from ordered data.
   - **Predictability**: The time component makes the identifiers more predictable compared to UUIDs, which are completely random. This can be advantageous for certain types of data processing and analytics.

2. **Compactness**:
   - **Storage Efficiency**: TSIDs are often more compact than UUIDs, making them more storage-efficient. This can lead to reduced storage costs and improved performance for indexing and searching.
   
3. **Improved Index Performance**:
   - **Clustered Index Optimization**: Since TSIDs are generated in a time-ordered manner, they fit well with clustered indexes, reducing the need for frequent rebalancing and thus improving insert performance.

### Comparison with UUIDs and ULIDs

- **UUIDs**: 
  - **Pros**: Universally unique, well-supported across various systems and platforms.
  - **Cons**: Poor database performance due to random nature, higher storage costs【9†source】.

- **ULIDs**:
  - **Pros**: Sortable, time-based component, same 128-bit length as UUIDs for compatibility, can improve insert performance and index maintenance【9†source】【12†source】.
  - **Cons**: While better than UUIDs, they still might require more space compared to more compact solutions like TSIDs.

- **TSIDs**:
  - **Pros**: More compact than UUIDs and ULIDs, time-based for sortability, efficient for database operations, predictable generation pattern.
  - **Cons**: Less widespread adoption and support compared to UUIDs, might require additional effort to integrate with existing systems.

### Implementation Considerations

- **Language and Framework Support**: Ensure that your development environment and ORM support TSIDs. This might involve using third-party libraries or writing custom code for integration.
- **Compatibility**: Evaluate the effort required to transition from UUIDs or ULIDs to TSIDs. This includes changes to database schema, application code, and any dependent systems or services.

### Conclusion

TSIDs can indeed be a better option than UUIDs and ULIDs in terms of performance and storage efficiency, particularly for applications requiring time-ordered data. However, the decision should be based on your specific requirements, including existing system compatibility and support for TSIDs in your technology stack.

For further insights, you may want to explore detailed comparisons and performance benchmarks from reliable sources and consider conducting your own tests to determine the best fit for your use case.

## TSIDs, using 64 bits, can be more efficient than ULIDs

You're correct that TSIDs, using 64 bits, can be more efficient than ULIDs, which use 128 bits. This efficiency makes TSIDs particularly appealing for certain applications, especially when considering database performance and keyset pagination.

### Key Differences and Benefits

1. **Bit Size and Storage Efficiency**:
   - **TSID (64 bits)**: Requires less storage space, which can lead to performance improvements, particularly in large databases. Smaller size means less data to transfer and index.
   - **ULID (128 bits)**: Offers a larger space for uniqueness, but at the cost of increased storage requirements.

2. **Natural Fit for Databases**:
   - **Bigint Compatibility**: TSIDs fit well into databases as they can be stored in a `BIGINT` column type, which is typically 64 bits. This makes them a natural choice for databases that optimize for `BIGINT` types, reducing overhead.
   - **Clustered Index Performance**: Since TSIDs are time-based and sorted, they work well with clustered indexes, leading to efficient insert operations and potentially faster query performance.

3. **Keyset Pagination**:
   - **TSIDs**: Their compact size and ordered nature make them well-suited for keyset pagination, which relies on sorted unique identifiers to paginate efficiently. This is particularly useful in applications requiring consistent and performant pagination over large datasets.

4. **Use Case Suitability**:
   - **ULIDs**: Better suited for distributed systems where uniqueness across systems is critical and 128-bit identifiers can be managed efficiently.
   - **TSIDs**: Ideal for systems where storage efficiency, database performance, and natural ordering are prioritized.

### Implementation and Adoption

- **.NET and Other Languages**: Libraries exist for both TSIDs and ULIDs in various programming languages, making it easier to implement them in your systems. For .NET, you can find TSID libraries that help integrate these identifiers into your applications.

### Practical Considerations

- **Migration**: Moving from UUIDs or ULIDs to TSIDs will require schema changes and data migration. Ensure that your system's architecture can handle this transition smoothly.
- **Indexing and Querying**: Adjust your indexing and querying strategies to take full advantage of TSIDs' ordered nature and storage efficiency.

### Conclusion

TSIDs offer compelling advantages in terms of storage efficiency and performance, especially when using a `BIGINT` column type for databases. They are well-suited for keyset pagination due to their compact size and natural ordering. If your primary concerns are database performance and storage, and you don't require the extensive uniqueness space provided by 128-bit identifiers, TSIDs can be an excellent choice.

For further insights and practical implementation guidance, refer to detailed resources and performance analyses on TSIDs and their integration into various systems.


## How do you sync up the node ids in a cluster environment ? Have you considered using a SQL table to allocate and free up the node_id ?

Synchronizing node IDs in a cluster environment is crucial to avoid conflicts and ensure that each node generates unique TSIDs. Here’s a detailed approach on how to achieve this, including the use of a SQL table for node ID allocation and freeing up node IDs upon node shutdown or crashes.

### Steps to Sync Node IDs

1. **Centralized Node ID Management**:
   - **SQL Table**: Create a table in your database to manage node IDs. This table will keep track of which node IDs are in use and which are available.
     ```sql
     CREATE TABLE NodeIDs (
         node_id INT PRIMARY KEY,
         in_use BOOLEAN NOT NULL DEFAULT FALSE,
         last_assigned TIMESTAMP NULL
     );
     ```

2. **Node ID Allocation**:
   - **Assigning Node ID**: When a node starts up, it queries the SQL table to find an available node ID and marks it as in use.
     ```sql
     UPDATE NodeIDs
     SET in_use = TRUE, last_assigned = CURRENT_TIMESTAMP
     WHERE node_id = (
         SELECT node_id
         FROM NodeIDs
         WHERE in_use = FALSE
         LIMIT 1
     )
     RETURNING node_id;
     ```

3. **Freeing Up Node IDs**:
   - **Graceful Shutdown**: When a node shuts down gracefully, it updates the SQL table to mark its node ID as available.
     ```sql
     UPDATE NodeIDs
     SET in_use = FALSE, last_assigned = NULL
     WHERE node_id = :current_node_id;
     ```
   - **Crash Detection**: To handle crashes, implement a heartbeat mechanism. Each node periodically updates a timestamp in the SQL table. If a node fails to update its timestamp within a certain threshold, it is considered crashed, and its node ID can be freed up.
     ```sql
     UPDATE NodeIDs
     SET in_use = FALSE, last_assigned = NULL
     WHERE node_id = :node_id
     AND last_assigned < (CURRENT_TIMESTAMP - INTERVAL 'your_threshold');
     ```

4. **Node Reassignment**:
   - **Reassigning Freed Node IDs**: After detecting a crash, the freed node IDs become available for new nodes or restarting nodes. The allocation query will pick up these IDs as part of its normal operation.

### Implementation Considerations

1. **Concurrency Control**:
   - Ensure that the SQL operations for allocating and freeing node IDs are atomic to avoid race conditions. Use transactions to ensure consistency.

2. **Monitoring and Alerts**:
   - Implement monitoring to track node health and the status of node ID assignments. Set up alerts to notify administrators of any issues, such as unassigned node IDs or nodes that have crashed.

3. **Fallback Mechanisms**:
   - Design fallback mechanisms in case the SQL database becomes unavailable. Nodes could temporarily use a local configuration to avoid complete failure but should sync back with the SQL table once the database is available.

### Example in Practice

Here’s an example of how this process could be integrated into a Node.js application using Sequelize (a promise-based Node.js ORM):

```javascript
const { Sequelize, DataTypes, Op } = require('sequelize');
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'postgres',
});

const NodeID = sequelize.define('NodeID', {
  node_id: { type: DataTypes.INTEGER, primaryKey: true },
  in_use: { type: DataTypes.BOOLEAN, defaultValue: false },
  last_assigned: { type: DataTypes.DATE, allowNull: true },
});

// Function to allocate a node ID
async function allocateNodeID() {
  await sequelize.transaction(async (t) => {
    const node = await NodeID.findOne({
      where: { in_use: false },
      lock: true,
      transaction: t,
    });
    if (node) {
      node.in_use = true;
      node.last_assigned = new Date();
      await node.save({ transaction: t });
      return node.node_id;
    }
    throw new Error('No available node IDs');
  });
}

// Function to free up a node ID
async function freeNodeID(node_id) {
  await NodeID.update(
    { in_use: false, last_assigned: null },
    { where: { node_id: node_id } }
  );
}

// Periodic heartbeat to update last_assigned
async function heartbeat(node_id) {
  await NodeID.update(
    { last_assigned: new Date() },
    { where: { node_id: node_id } }
  );
}

// Example usage
(async () => {
  try {
    const nodeId = await allocateNodeID();
    console.log(`Node ID allocated: ${nodeId}`);
    // Set up heartbeat
    setInterval(() => heartbeat(nodeId), 60000); // Update every minute
  } catch (error) {
    console.error('Failed to allocate node ID:', error);
  }
})();
```

### Conclusion

Using a SQL table to manage node ID allocation and deallocation in a cluster environment ensures that node IDs are unique and efficiently utilized. This approach provides a robust solution for maintaining node ID consistency, even in the case of node crashes or restarts. Implementing such a system requires careful consideration of concurrency, monitoring, and failover mechanisms to ensure smooth operation.

## The nodeId can be set as an environment variable, so each node gets its own unique identifier.

Setting the node ID as an environment variable ensures that each node in a cluster gets its unique identifier. This approach simplifies the management of node IDs and reduces the complexity involved in dynamically assigning and tracking node IDs in a database. Here's a detailed explanation of how this setup works and its advantages:

### Static Node ID Assignment

1. **Environment Variable Configuration**:
   - **Unique Node IDs**: Each node in the cluster is assigned a unique node ID through an environment variable. This can be configured in the deployment scripts or directly in the environment where the node runs.
   - **Configuration Example**:
     ```sh
     export NODE_ID=1
     ```

2. **Integration with TSIDs**:
   - **TSID Generation**: The node ID is used as part of the TSID generation process. Since the node ID is set at startup and remains constant, it ensures that each node generates unique identifiers without conflicts.
   - **Node Configuration**: The application reads the node ID from the environment variable during startup.
     ```javascript
     const nodeId = process.env.NODE_ID;
     ```

### Benefits of Static Node ID Assignment

1. **Simplicity**:
   - **Reduced Complexity**: Using environment variables to set node IDs eliminates the need for a centralized system to manage node IDs dynamically. This simplifies the overall system architecture.
   - **Easy Management**: Environment variables are easy to manage and configure, especially in containerized environments like Docker or orchestration systems like Kubernetes.

2. **Reliability**:
   - **Consistent IDs**: Each node consistently uses the same node ID, reducing the risk of ID conflicts and ensuring predictable identifier generation.
   - **Crash Recovery**: In case of a node crash, the environment variable ensures that the node retains its ID upon restart, avoiding potential issues with ID reassignment.

3. **Scalability**:
   - **Horizontal Scaling**: As new nodes are added to the cluster, each can be assigned a unique node ID via environment variables. This method scales efficiently without additional complexity.
   - **Dynamic Allocation (Optional)**: For environments that require dynamic allocation, a lightweight service or startup script can assign unique node IDs before starting the main application, updating the environment variable accordingly.

### Example: Using Node.js and Environment Variables

Here’s how you can set up a Node.js application to use TSIDs with a static node ID:

1. **Setting the Node ID**:
   - In a Dockerfile or deployment script:
     ```dockerfile
     ENV NODE_ID=1
     ```

   - In Kubernetes:
     ```yaml
     env:
       - name: NODE_ID
         value: "1"
     ```

2. **Application Code**:
   - Read the node ID and use it in TSID generation:
     ```javascript
     const TSID = require('tsid-library'); // hypothetical library
     const nodeId = parseInt(process.env.NODE_ID, 10);

     function generateTSID() {
       return TSID.generate({ nodeId });
     }

     console.log(generateTSID());
     ```

### Conclusion

Using environment variables to set node IDs in a cluster environment offers a simple, reliable, and scalable solution for generating unique TSIDs. This approach leverages the static assignment of node IDs, reducing the need for dynamic management and ensuring consistent identifier generation across the cluster. This method is particularly advantageous in modern deployment environments where configuration management is streamlined and automated.

## References

1. https://vladmihalcea.com/uuid-database-primary-key/
2. https://db-engines.com/en/ranking
3. https://www.cybertec-postgresql.com/en/unexpected-downsides-of-uuid-keys-in-postgresql/
4. https://buildkite.com/blog/goodbye-integers-hello-uuids
5. https://gist.github.com/beginor/9d9f90bc58e1313f6aecd107f8296732
6. https://en.wikipedia.org/wiki/Universally_unique_identifier#Versions
7. https://twitter.com/vlad_mihalcea/status/1609113833037090816
8. https://www.baeldung.com/java-generating-time-based-uuids
9. https://stackoverflow.com/questions/18244897/how-to-generate-time-based-uuids
10. https://github.com/cowtowncoder/java-uuid-generator
11. https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-uuid/
12. https://github.com/eugenp/tutorials
13. https://twitter.com/mjovanovictech/status/1802952859085861078
14. https://twitter.com/mjovanovictech/status/1802952859085861078
15. 
