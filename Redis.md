# Redis

Fast Single-Threaded Redis

## Why is Redis Fast?

Redis is fast for in-memory data storage. Its speed has made it popular for caching, session storage, and real-time analytics. But what gives Redis its blazing speed? Let's explore:

## RAM-Based Storage

At its core, Redis primarily uses main memory for storing data. Accessing data from RAM is orders of magnitude faster than from disk. This is a major reason for Redis's speed.

However, RAM is volatile. To persist data, Redis supports disk snapshots and append-only file logging. This combines RAM's performance with disk's permanence.

There is a tradeoff though - recovery from disk is slow. If a Redis instance fails, restarting from disk can be slow compared to failing over to a replica instance fully in memory. So while Redis offers durability via disk, it comes at the cost of slower recovery.

A better solution is Redis replication. With a synchronized replica kept in memory, failover is instant with no rehydration. This maintains speed and near-instant recovery.

## IO Multiplexing & Single-threaded Read/Write

Redis uses an event-driven, single-threaded model for its core operations. A main event loop handles all client requests and data operations sequentially. This single-threaded execution avoids context switching and synchronization overhead typical of multi-threaded systems.

Redis uses non-blocking I/O to handle multiple connections asynchronously. This allows it to support many client connections with very low overhead,

Redis does leverage threading in certain areas:

- Background tasks like taking snapshots.
- I/O threads are used for certain operations.
- Modules can use threads.
- Since Redis 6.0, it supports multi-threaded I/O for network communication, improving performance on multi-core systems.

Redis also uses pipelining for high throughput. Clients pipeline commands without waiting for each response. This allows more efficient network round trips, boosting overall performance.

## Efficient Data Structures

Redis supports various optimized data structures, from linked lists, zip lists, and skip lists to sets, hashes, and sorted sets, among others. Each is carefully designed for specific use cases for quick and efficient data access.

Over to you: With Redis now supporting some multi-threading, how should we configure it to fully utilize all the CPU cores of modern hardware when deploying in production?

![Redis](./img/Databases/redis.jpg)

## Why is single-threaded Redis so fast?

Redis is able to achieve high performance due to its in-memory data storage, which allows it to access data much faster than if it were stored on disk. Additionally, Redis is single-threaded, which means that it only uses one core of a CPU at a time. This allows Redis to avoid the overhead associated with thread management and synchronization, which can slow down multi-threaded systems. Additionally, Redis uses a non-blocking I/O model, which means that it can handle multiple requests simultaneously without having to wait for one to complete before processing the next. This also contributes to its high performance.

## What fundamental design decisions did the developers make more than a decade ago that stood to test of time?

There are several fundamental design decisions made by the Redis developers more than a decade ago that have stood the test of time:

1. In-memory data storage: Redis stores all data in memory, which allows for much faster access times than disk-based storage. This design decision was made to optimize performance, and it has proven to be an effective way to handle real-time data processing.

2. Single-threaded architecture: Redis is a single-threaded application, which allows it to avoid the overhead associated with thread management and synchronization. This design decision contributes to Redis's high performance and has proven to be effective for handling high-throughput workloads.

3. Data structures: Redis supports a variety of data structures, such as strings, lists, sets, and hashes, which makes it a very versatile and flexible data store. This design decision has allowed Redis to be used in a wide range of use cases, including caching, messaging, and real-time analytics.

4. Support for persistence: Redis provides a persistence feature, which allows it to save data to disk. This design decision allows Redis to survive a system reboot and maintain data even if the process crashes.

5. Networking: Redis is designed to work over a network, which allows it to be easily deployed in distributed systems. This design decision enables Redis to handle high-availability and high-scalability workloads, making it a popular choice for building large-scale, distributed systems.

All of these design decisions made by the Redis developers more than a decade ago are still relevant today and have stood the test of time as they are still widely used today in various applications.

## Why would a single threaded design lead to high performance? Wouldn't it be faster if it uses threads to leverage all the CPU cores?

A single-threaded design can lead to high performance because it avoids the overhead associated with thread management and synchronization. In a multi-threaded design, the operating system needs to manage and schedule multiple threads, which can introduce additional overhead and slow down the system. Additionally, in a multi-threaded design, the threads may need to synchronize access to shared data structures, which can also introduce overhead and slow down the system.

In contrast, a single-threaded design only needs to manage one thread, which reduces the overhead associated with thread management and synchronization. Additionally, a single-threaded design can take full advantage of the CPU's cache, which can further improve performance.

However, it's not always the case that a single-threaded design is faster than a multi-threaded design. A multi-threaded design can take full advantage of multiple CPU cores, which can significantly improve performance for workloads that are computationally intensive. Additionally, a multi-threaded design can also improve performance for workloads that are IO-bound by allowing multiple requests to be handled simultaneously.

Redis is designed to handle high-throughput workloads, which are often IO-bound. Therefore, a single-threaded design with a non-blocking I/O model allows Redis to handle many requests simultaneously, without having to wait for one to complete before processing the next. This makes Redis faster than a multi-threaded design in this specific use case. Additionally, as Redis stores all the data in memory, the CPU cores are not limited by the disk I/O speed and can handle the requests quickly.


## How does a single threaded codebase handle many thousands of incoming requests and outgoing responses at the same time? Won't the thread get blocked waiting for the completion of each request individually?

A single-threaded codebase can handle many thousands of incoming requests and outgoing responses at the same time by using non-blocking I/O. Non-blocking I/O allows the single thread to handle multiple requests simultaneously without having to wait for one to complete before processing the next.

In a traditional, blocking I/O model, when a request is received, the thread will block and wait for the request to be fully processed before moving on to the next request. This can lead to poor performance when handling a high number of requests.

In contrast, a non-blocking I/O model allows the thread to initiate a request and then move on to handling the next request, without having to wait for the previous request to complete. When the previous request completes, the thread is notified and can then process the response. This allows the thread to handle multiple requests simultaneously, without getting blocked on any individual request.

Redis uses a non-blocking I/O model, which allows it to handle multiple requests simultaneously. Additionally, Redis uses a single-threaded event loop, which is a programming construct that allows a single thread to handle multiple events (such as incoming requests) in a non-blocking way. The event loop continuously monitors for new events, and as soon as an event is detected, it's passed to the event handler (request handler in this case) which process the event and sends the response. This way, Redis can handle thousands of requests and responses simultaneously, without getting blocked on any individual request.

## where I/O multiplexing comes?

I/O multiplexing is a technique that allows a single thread to handle multiple I/O operations simultaneously. It is often used in conjunction with non-blocking I/O to improve the performance of network-based applications.

There are two main types of I/O multiplexing: select and poll. 

- Select: It allows the application to monitor multiple file descriptors, and it blocks the execution of the process until one or more of the file descriptors become ready for I/O. It's not very efficient for a large number of file descriptors.

- Poll: It is similar to select, but it is more efficient for a large number of file descriptors. It allows the application to monitor multiple file descriptors, but it doesn't block the execution of the process. Instead, it returns immediately with the status of the file descriptors that are ready for I/O.

I/O multiplexing is used in Redis to monitor multiple file descriptors, such as incoming network connections or disk I/O operations, simultaneously. The event loop in Redis uses poll to monitor these file descriptors. When an event (e.g. incoming connection) is detected, the event loop passes it to the corresponding event handler (connection handler in this case) that processes the event and sends the response.

By using I/O multiplexing, Redis is able to handle multiple I/O operations simultaneously, without getting blocked on any individual operation. This allows Redis to handle a high number of incoming requests and outgoing responses in a single-threaded environment.

## is it not uncommon to have several Redis instances running on a single server to utilize more CPU cores?

Yes, it is not uncommon to have several Redis instances running on a single server to take advantage of multiple CPU cores. This is known as running multiple Redis instances on the same physical machine.

Each Redis instance runs in its own process, and since Redis is single-threaded, each process only uses one core of the CPU. By running multiple Redis instances on the same physical machine, each instance can use a different core of the CPU, which allows the overall system to take full advantage of all the available cores.

This approach can be beneficial when the workload requires more computational power than a single Redis instance can provide. However, it's important to keep in mind that running multiple Redis instances on the same physical machine can also increase the system's memory usage. As Redis stores all the data in memory, each instance will consume memory, so it's important to have enough memory to run all the instances.

Additionally, it's also important to consider the network bandwidth and disk I/O when running multiple instances on the same machine as each instance will compete for the same resources.

There are also some other approaches to scale Redis such as using Redis Cluster, Redis Sentinel, Redis Shard or using Redis proxies like twemproxy, nutcracker, codis, etc. These approaches are to scale out the Redis horizontally, instead of running multiple Redis instances on the same machine which is a vertical scaling.

## top 5 use cases where it shines: 
 
𝗖𝗮𝗰𝗵𝗶𝗻𝗴 
The most common use case is to utilize Redis for caching objects or pages. This helps protect the database layer from overloading. 
 
𝗦𝗲𝘀𝘀𝗶𝗼𝗻 
We use Redis to share user session data among stateless servers. 
 
𝗗𝗶𝘀𝘁𝗿𝗶𝗯𝘂𝘁𝗲𝗱 𝗹𝗼𝗰𝗸 
We use Redis distributed locks to grant mutually exclusive access to shared resources. 
 
𝗖𝗼𝘂𝗻𝘁𝗲𝗿 𝗮𝗻𝗱 𝗥𝗮𝘁𝗲 𝗟𝗶𝗺𝗶𝘁𝗲𝗿 
These two use cases are related. 
 
We use Redis to track like counts on social media apps, and also to enforce rate limits on our endpoints. 
 
𝗟𝗲𝗮𝗱𝗲𝗿𝗯𝗼𝗮𝗿𝗱 
Sorted Sets is a delightful way to implement a gaming leaderboard. 

![Top 5 Redis Use Cases](./img/x/sahnlam/Redis%20Use%20Cases.jpg)

## Video

 * [What is Redis and What Does It Do?](https://www.youtube.com/watch?v=8A_iNFRP0F4)
	> [<img src="https://img.youtube.com/vi/8A_iNFRP0F4/0.jpg" width="200">](https://www.youtube.com/watch?v=8A_iNFRP0F4 "What is Redis and What Does It Do? by CBT Nuggets 232,635 views 6 minutes, 47 seconds")
 * [Redis Crash Course - the What, Why and How to use Redis as your primary database](https://www.youtube.com/watch?v=OqCK95AS-YE)
	> [<img src="https://img.youtube.com/vi/OqCK95AS-YE/0.jpg" width="200">](https://www.youtube.com/watch?v=OqCK95AS-YE "Redis Crash Course - the What, Why and How to use Redis as your primary database by TechWorld with Nana 398,668 views 23 minutes")
 * [I&#39;ve been using Redis wrong this whole time...](https://www.youtube.com/watch?v=WQ61RL1GpEE)
	> [<img src="https://img.youtube.com/vi/WQ61RL1GpEE/0.jpg" width="200">](https://www.youtube.com/watch?v=WQ61RL1GpEE "I&#39;ve been using Redis wrong this whole time... by Dreams of Code 307,803 views 20 minutes")
 * [I&#39;ve been using Redis wrong this whole time...](https://www.youtube.com/watch?v=WQ61RL1GpEE)
	> [<img src="https://img.youtube.com/vi/WQ61RL1GpEE/0.jpg" width="200">](https://www.youtube.com/watch?v=WQ61RL1GpEE "I&#39;ve been using Redis wrong this whole time... by Dreams of Code 307,803 views 20 minutes")
 * [I&#39;ve been using Redis wrong this whole time...](https://www.youtube.com/watch?v=WQ61RL1GpEE)
	> [<img src="https://img.youtube.com/vi/WQ61RL1GpEE/0.jpg" width="200">](https://www.youtube.com/watch?v=WQ61RL1GpEE "I&#39;ve been using Redis wrong this whole time... by Dreams of Code 307,803 views 20 minutes")
 * [Top 5 Redis Use Cases](https://www.youtube.com/watch?v=a4yX7RUgTxI)
	> [<img src="https://img.youtube.com/vi/a4yX7RUgTxI/0.jpg" width="200">](https://www.youtube.com/watch?v=a4yX7RUgTxI "Top 5 Redis Use Cases by ByteByteGo 150,548 views 6 minutes, 28 seconds")
 * [System Design: Why is single-threaded Redis so fast?](https://www.youtube.com/watch?v=5TRFpFBccQM)
	> [<img src="https://img.youtube.com/vi/5TRFpFBccQM/0.jpg" width="200">](https://www.youtube.com/watch?v=5TRFpFBccQM "System Design: Why is single-threaded Redis so fast? by ByteByteGo 275,539 views 3 minutes, 39 seconds")

## References

1. http://www.devmedia.com.br/redis-e-java-revista-java-magazine-114/27504