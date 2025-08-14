# Ultimate Database Selection Guide

## **Ultimate Database Selection Guide (Extended)**

| **Data Type**            | **Use Case**                    | **Category**   | **AWS**                        | **Azure**             | **GCP**                  | **Cloud Agnostic / Additional**                                  |
| ------------------------ | ------------------------------- | -------------- | ------------------------------ | --------------------- | ------------------------ | ---------------------------------------------------------------- |
| 🔵 **Structured**        | 📊 ACID Transactions (OLTP)     | Relational     | RDS                            | Azure SQL Database    | Cloud SQL, Cloud Spanner | SQL Server, Oracle, CockroachDB, MySQL, **MariaDB**, PostgreSQL  |
|                          | 📈 Analytics (OLAP)             | Columnar       | RedShift                       | Azure Synapse         | BigQuery                 | Snowflake, Databricks, HIVE, **ClickHouse**, Vertica, Greenplum  |
|                          | 🗄 Dictionary / Key-Value       | Key-Value      | DynamoDB                       | CosmosDB              | BigTable                 | Redis, RocksDB, Ignite, **Aerospike**, Riak KV                   |
|                          | ⚡ Cache (In-memory)             | In-memory      | ElastiCache                    | Azure Cache for Redis | Memorystore              | Redis, Memcached, Hazelcast, Ignite                              |
|                          | 🗄 2-D Key-Value                | Wide Column    | Keyspaces                      | CosmosDB              | BigTable                 | HBase, Cassandra, ScyllaDB                                       |
| 🟢 **Semi-Structured**   | ⏱ Time Series                   | Timeseries     | Timestream                     | CosmosDB              | BigTable, BigQuery       | TimescaleDB, OpenTSDB, InfluxDB, **Prometheus**, VictoriaMetrics |
|                          | 📐 Mathematical Representation  | Vector         | —                              | —                     | —                        | Milvus, Pinecone, Weaviate, Qdrant, FAISS                        |
|                          | 📍 Location & Geo-entities      | Geospatial     | Keyspaces                      | CosmosDB              | BigTable, BigQuery       | Solr, PostGIS, MongoDB (GeoJSON), Carto                          |
|                          | 🔗 Entity Relationships         | Graph          | Neptune                        | CosmosDB              | JanusGraph Table         | OrientDB, Neo4j, TigerGraph, **ArangoDB**                        |
|                          | 📄 Nested Objects (XML/JSON)    | Document       | DocumentDB                     | CosmosDB              | Firestore                | MongoDB, Couchbase, **CouchDB**, RavenDB                         |
| 🟣 **Semi/Unstructured** | 🔍 Text Search                  | Text Search    | OpenSearch, CloudSearch        | Cognitive Search      | Search APIs on Database  | Elasticsearch, Solr, Elassandra, **Meilisearch**, Typesense      |
| 🟣 **Unstructured**      | 🗂 Blob Storage                 | Blob           | S3                             | Blob Storage          | Cloud Storage            | HDFS, MinIO, Ceph, GlusterFS                                     |
| 🆕 **New**               | 📡 Streaming / Event Storage    | Event Store    | Kinesis                        | Event Hubs            | Pub/Sub                  | Apache Kafka, Redpanda, Pulsar, EventStoreDB                     |
| 🆕 **New**               | 📜 Ledger / Blockchain          | Ledger DB      | Quantum Ledger Database (QLDB) | —                     | —                        | Hyperledger Fabric, Corda, BigchainDB                            |
| 🆕 **New**               | 🔄 Multi-Model DB               | Multi-Model    | —                              | —                     | —                        | ArangoDB, OrientDB, MarkLogic                                    |
| 🆕 **New**               | 📊 Search + Analytics Hybrid    | Search-OLAP    | OpenSearch (with aggregations) | —                     | —                        | Elasticsearch, Apache Solr, Apache Druid                         |
| 🆕 **New**               | 📱 Edge / Embedded DB           | Embedded       | —                              | —                     | —                        | SQLite, Realm, RocksDB                                           |
| 🆕 **New**               | ⚡ Real-time Analytics / HTAP    | HTAP           | —                              | —                     | —                        | TiDB, SingleStore (MemSQL), YugabyteDB                           |
| 🆕 **New**               | 🗂 Object & File Metadata Store | Metadata Store | AWS Glue Data Catalog          | —                     | Dataplex                 | Apache Ozone, Databricks Unity Catalog                           |
| 🆕 **New**               | 🤖 AI/ML Feature Store          | Feature Store  | SageMaker Feature Store        | —                     | Vertex AI Feature Store  | Feast, Tecton                                                    |

---


