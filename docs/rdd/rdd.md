# Resilient Distributed Datasets: A Fault-Tolerant Abstraction for In-Memory Cluster Computing

*A distributed memory abstraction that lets programmers perform in-memory computations on large clusters in a fault-tolerant manner.*

## Background

- Current distributed computing frameworks(like MapReduce) lack abstractions for leveraging distributed memory, so they are inefficient for applications that need to reuse intermediate results(iterative machine learning, interactive data mining, etc).
- Current abstractions for cluster in-memory storage(like Piccolo) provide interface with fine-grained updates. They have to replicate data or log updates across machines for fault tolerance, which is expensive for data-intensive workloads.

## RDD Model

- An RDD is a read-only, partitioned collection of records. RDDs can only be created through deterministic operations on (1) data in stable storage or (2) other RDDs.
- RDD provides interface based on coarse-grained transformations that apply the same operation to many data items, which allows efficient fault tolerance by logging the transformations used to build a dataset(lineage) rather than actual data.
- RDD is not suitable for applications that needs to make asynchronous fine-grained updates to shared states(incremental web crawler, etc).

![rdd vs dsm](images/rdd vs dsm.jpg)

## Implementation

- An RDD is represented as (1) a set of partitions, (2) a set of dependencies of parent RDDs, (3) a function for computing the dataset from parents and (4) metadata about partitioning scheme and data placement.
- There are two types of dependencies, narrow(like map or filter) and wide(like join). Narrow dependencies can be more efficiently executed and recovered after failure.
- The scheduler examines the RDD's lineage graph to build a DAG of stages to execute. Each stage contains as many pipelined transformations with narrow dependencies as possible. The boundaries of stages are shuffle operations or any already computed partitions.
- The scheduler assigns tasks to machines based on data locality using delay scheduling. A task is sent to node that has the partition available in memory or is preferred location for HDFS. If a task fails, the scheduler rerun it on another node and resubmit tasks for parent partitions if necessary.
- Use LRU eviction policy at the level of RDDs to manage the limited memory available.
- Checkpointing is useful for RDDs with long lineage graphs containing wide dependencies. The read-only nature of RDDs make them simpler to checkpoint than distributed shared memory.