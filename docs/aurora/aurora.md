# Amazon Aurora: Design Considerations for High Throughput Cloud-Native Relational Databases

*A cloud-native relational database service for OLTP workloads.*

## Introduction

- We believe the central constraint in high throughput data processing has moved from compute and storage to the network.
- Aurora uses a novel architecture with a fleet of database instances and storage service. Several database functions(redo logging, crash recovery, etc) are offloaded to the storage service, which is like a virtualized segmented redo log (shared-disk architecture).
- **Key idea:** the log is the database; any page that the storage system materializes are simply a cache of log application.

## Architecture

- To tolerate AZ failure, Aurora replicates each data item 6 ways across 3AZs with 2 copies in each AZ.
- Database volume is partitioned into 10GB segments. Each segment is replicated 6 times into a Protection Group.
- The only writes that cross the network are redo log records, so network load is drastically reduced despite amplifying write for replication. 
- Storage nodes gossips with peers to fill gaps in received log records. Durable log application happens at the storage nodes continuously and  asynchronously.
- Each log record has a monotonically-increasing Log Sequence Number(LSN).
- Instead of 2PC protocol, Aurora maintains points of consistency and durability and advances them when receiving acknowledgements for storage requests.
  - Durability: the highest LSN at which all prior log records are available.
  - Consistency: each transaction is broken up to mini-transactions, and the final log record in a mini-transaction is a consistency point.
- Normally  read quorum is not needed since the database feeds log records to storage nodes and tracks progress.

![architecture](images/architecture.png):

## Extra Notes

- In shared-disk architecture, all data is shared by all nodes. In shared-nothing architecture, each node manages a subset of data.
- It is hard to use change buffer in shared-disk architecture, so writes are penalized when there are secondary indexes.