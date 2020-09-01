# Gorilla: A Fast, Scalable, In-Memory Time Series Database

*Facebook's in-memory time series database for monitoring.*

## Introduction

- Gorilla functions as a write-through cache for most recent monitoring data written to a HBase datastore.
- **Key Insight:** aggregate analysis is more important than individual data points; recent data points are more important than older ones.
- Gorilla optimizes for high read/write availability at the expense of possibly dropping data on the write path.

## Data Model and Compression

- Each monitoring data point is a 3-tuple of a string key (time series identifier), an int64 timestamp and a double value.
- Gorilla compresses data points within a time series. Timestamps are values are compressed separately.
  - For timestamps, since most data points arrive at a fixed interval, Gorilla encodes delta-of-deltas with variable length encoding.
  - For values, since most data points don't change significantly when compared to neighbors, Gorilla encodes XOR'd values with variable length encoding.
- Time series datasets are sharded by keys. A Gorilla node will own multiple shards (shared-nothing, horizontal scaling).

## Architecture

### In-Memory

- A node stores the mappings of $shard\_id \rightarrow(name\rightarrow time\_series)$​.
- A time series data structure has a sequence of immutable closed blocks for old data and one append-only open block for most recent data. When requested, the data blocks are returned without decompression via RPCs.
- Concurrency is attained by per-shard read-write spin locks and per-time-series spin locks.

### On-Disk

- For each shard, the node stores an append-only log and complete block files for checkpointing in a directory.
- The log file is not WAL -- data is buffered up to 64MB before being flushed. Gorilla trades off potential data loss for higher availability for writes.
- Data is persisted on a distributed file system with replication.

### Handling Failures

- Gorilla maintains two hosts in separate geographical regions with fail-over. On a write, data is streamed to each host without attempt to guarantee consistency.
- Within a region, a Paxos-based ShardMaster assigns shards to nodes and move shards when a node fails. During shard reassignment, the write clients buffer up to 1 minute of most recent data points and discard older ones.
- For incomplete data, Gorilla returns partial results and lets caller decide how to proceed.