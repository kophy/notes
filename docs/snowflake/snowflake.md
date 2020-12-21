# The Snowflake Elastic Data Warehouse

*An enterprise-ready data warehousing solution for the cloud.*

## Background

- Traditional data warehousing solutions are not able to leverage the cloud's elasticity and struggle with semi-structure data and unpredictable workloads.
- Big data platforms like Hadoop/Spark require significant engineering work to use.

## Architecture

- Pure shared-nothing architecture tightly couples compute and storage resources, so it cannot well handle heterogenous workloads (I/O bandwidth vs compute), frequent membership changes (data has to be reshuffled) and online upgrades. 
- Snowflake uses the *multi-cluster, shared-data* architecture, which separates storage and compute into loosely-coupled, independently scalable services:
  - **Data storage** stores table data, query temp data and results in S3. Tables are horizontally partitioned into large, immutable files in PAX format (S3 allows GET requests over parts of files, so virtual warehouses only needs to download the file headers and interested columns).
  - **Virtual warehouse** consists of a cluster of EC2 machines. Each worker node spawns processes to run queries and maintains a disk-based LRU cache for table data.
    - To improve hit rate and avoid redundant caching, the query optimizer assigns input files to worker nodes with consistent hashing (when membership changes, data shuffling is done lazily via cache replacement).
    - The execution engine is columnar, vectorized and push-based.
    - Snowflake uses file stealing to handle straggler nodes. 
  - **Cloud services** (access control, query optimizer, transaction manager, etc) are stateless and shared between users. Metadata (table schemas, mappings between tables to S3 files, etc) is stored in a global key-value store.
- Instead of indices, Snowflake uses min-max based pruning (the system maintains data distribution information for data chunks) to access only relevant data chunks.

![architecture](images/architecture.png)

## Feature Highlights

- Snowflake provides a pure SAAS experience; no tuning knobs, no physical design, etc.
- Snowflake uses techniques like replication, stateless services and metadata mapping layer to ensure continuous availability.
- For semi-structured data, Snowflake performs automatic type inference via statistical analysis. To optimize query performance for complex paths, Snowflake keeps bloom filters for paths (not values!) present in a data chunk.
- Time travel and cloning can be efficiently implemented since Snowflake uses multi-version concurrency control (MVCC).
- Snowflake provides a hierarchy key model and uses key rotation and rekeying to ensure standardized life cycles.