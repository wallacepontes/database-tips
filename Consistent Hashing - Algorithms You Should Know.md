# Consistent Hashing | Algorithms You Should Know

Consistent hashing is a technique used in distributed systems to distribute data across a cluster of nodes in a way that minimizes reorganization when nodes are added or removed. This is particularly useful for load balancing and managing data in a scalable manner. Here are some key algorithms and concepts related to consistent hashing that you should know:

### 1. **Basic Consistent Hashing**
- **Ring Hashing:** Data and nodes are mapped to points on a circular hash space. Each data point is stored on the first node encountered when moving clockwise around the ring from the data point's position.
- **Hash Function:** A good hash function (e.g., MD5, SHA-1) is used to distribute the data and nodes uniformly around the ring.

### 2. **Virtual Nodes**
- **Virtual Nodes (vnodes):** To improve load distribution and reduce the impact of node failure, each physical node is assigned multiple virtual nodes. Each vnode is treated as an independent node in the hash ring.
- **Load Balancing:** Virtual nodes help in balancing the load more evenly as each physical node is responsible for multiple parts of the hash ring.

### 3. **Consistent Hashing with Bounded Loads**
- **Bounded Load Consistent Hashing:** Ensures that the load on each node does not exceed a certain factor of the average load. This is achieved by adjusting the number of virtual nodes or using different algorithms like multi-probe consistent hashing.

### 4. **Rendezvous Hashing (Highest Random Weight, HRW)**
- **HRW:** Each key is assigned to the node with the highest hash value from a set of hash functions. This approach ensures minimal disruption when nodes are added or removed and allows for more flexible load balancing.

### 5. **Jump Consistent Hashing**
- **Jump Hash:** A fast and efficient algorithm that maps a key to one of a fixed number of buckets (nodes). It uses a series of jumps to quickly find the appropriate bucket, making it suitable for high-performance applications.

### 6. **Maglev Hashing**
- **Maglev Hash:** Used by Google for load balancing in their network. It achieves consistent hashing with minimal disruption using a permutation-based approach to distribute traffic evenly across backend servers.

### 7. **Ketama Hashing**
- **Ketama:** A specific implementation of consistent hashing used by the Memcached distributed caching system. It uses a continuum of hash values to ensure even distribution of keys across cache nodes.

### 8. **Chord Protocol**
- **Chord:** A protocol for peer-to-peer distributed systems that uses consistent hashing to assign keys to nodes. It supports efficient lookups and dynamically handles node arrivals and departures.

### Practical Applications
- **Distributed Databases:** Used in systems like Apache Cassandra, Amazon DynamoDB, and Redis Cluster to distribute data across nodes.
- **Content Delivery Networks (CDNs):** Ensures content is served from the nearest node to minimize latency.
- **Load Balancers:** Distributes incoming requests to backend servers in a balanced manner.
- **Caching Systems:** Used in systems like Memcached and Couchbase to distribute cache entries across multiple nodes.

Understanding these algorithms and their applications can help in designing and maintaining scalable and resilient distributed systems.

![Consistent Hashing](./img/x/sahnlam/Consistent%20hashing.jpg)

## the concept of immutability in consistent hashing is integral to its effectiveness in load balancing

Absolutely, consistent hashing plays a crucial role in load balancing, especially in distributed systems where data and requests need to be distributed evenly across multiple nodes. Immutability, in this context, refers to the ability to minimize the amount of re-distribution of data when changes occur in the system, such as adding or removing nodes. This is essential for maintaining a balanced load and ensuring the system's efficiency and reliability. Here’s how consistent hashing achieves this:

### Immutability and Load Balancing in Consistent Hashing

1. **Minimal Data Movement:**
   - When a node is added or removed in a consistent hashing setup, only a small subset of the data needs to be reallocated. This is because each node is responsible for a specific range of the hash space, and changes affect only the data within that range.
   - For example, in a hash ring with nodes and data points, adding a new node will only move the data that falls into the new node's hash range. The rest of the data remains undisturbed.

2. **Uniform Distribution:**
   - Consistent hashing aims to distribute data uniformly across the nodes. By doing so, it ensures that no single node becomes a bottleneck or gets overloaded with requests, which is critical for load balancing.
   - The use of virtual nodes further enhances this uniformity, as each physical node handles multiple ranges of the hash space, distributing the load more evenly.

3. **Scalability:**
   - As the system grows, consistent hashing allows for the seamless addition of new nodes without significant data re-distribution. This immutability property ensures that scaling up the system is efficient and does not disrupt the existing load distribution.

4. **Fault Tolerance:**
   - In case of node failures, consistent hashing reassigns the data from the failed node to the remaining nodes with minimal disruption. This resilience is crucial for maintaining balanced load and system availability.

### Example Use Case: Distributed Caching

In distributed caching systems like Memcached or Redis Cluster, consistent hashing helps in balancing the cache entries across multiple nodes:

- When a new cache node is added to the cluster, consistent hashing ensures that only a portion of the cache entries need to be moved to the new node. The majority of the cache entries remain on their original nodes, maintaining cache efficiency.
- Similarly, if a cache node fails, only the entries that were assigned to that node need to be redistributed, minimizing the impact on the overall system performance.

### Example Use Case: Distributed Databases

In distributed databases like Apache Cassandra or Amazon DynamoDB, consistent hashing ensures balanced data distribution and efficient scaling:

- Adding a new database node involves reassigning only a small fraction of the data partitions to the new node. This keeps the database balanced and reduces the overhead of data movement.
- Removing a node or handling node failures follows the same principle, with minimal data reallocation required, thus maintaining system stability and performance.

In summary, the concept of immutability in consistent hashing is integral to its effectiveness in load balancing. By minimizing data movement and ensuring uniform distribution, consistent hashing supports scalable, reliable, and efficient distributed systems.


## Videos

 * [Consistent Hashing | Algorithms You Should Know #1](https://www.youtube.com/watch?v=UF9Iqmg94tk)
	> [<img src="https://img.youtube.com/vi/UF9Iqmg94tk/0.jpg" width="200">](https://www.youtube.com/watch?v=UF9Iqmg94tk "Consistent Hashing | Algorithms You Should Know #1 by ByteByteGo 1,371 views 43 minutes")
* [Consistent Hashing - A Common Mistake in Choosing Partitioning Keys](https://www.youtube.com/watch?v=sLbOz2QBZgc)
	> [<img src="https://img.youtube.com/vi/sLbOz2QBZgc/0.jpg" width="200">](https://www.youtube.com/watch?v=sLbOz2QBZgc "Consistent Hashing - A Common Mistake in Choosing Partitioning Keys")

## References
1. https://en.wikipedia.org/wiki/Akamai_Technologies#cite_note-nuggets-7
2. Discord
3. Cassandra
4. DynamoDB
5. 