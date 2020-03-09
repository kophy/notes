# MapReduce: Simplified Data Processing on Large Clusters

*A programming model and an associated implementation for processing and generating large data sets.*

## Introduction

- Many computation tasks are conceptually straightforward, but given the size of input data, the computations have to be distributed across machines to finish in a reasonable amount of time.
- MapReduce is an abstraction that can express many computation tasks while hiding details of parallelization, fault-tolerance, data distribution and load balancing.
- Users specify a map function that processes a key/value pair to generate a set of intermediate key/value pairs, and a reduce function that merges all intermediate values associated with the same intermediate key.

## Execution

- A single master assigns tasks to workers; there are M map tasks and R reduce tasks in total.
- For map task, worker reads input, applies user-defined Map function and periodically writes intermediate results buffered in memory to local disk.
- For reduce task, worker uses rpcs to read intermediate results on map workers' local disks, sorts intermediate results to group occurrences of the same key, applies user-defined Reduce function and writes final results to a global file system.
- Master is responsible of propagating the locations of intermediate files from map tasks to reduce tasks.

![execution](images/execution.jpg)

## Fault Tolerance

- For worker failure, master periodically pings workers and marks the worker that has no response for a certain amount of time as failed. Completed map tasks and any in-progress tasks on that worker are rescheduled to other workers; no need to re-execute completed reduce tasks because output is stored in global file system instead of worker's local disk.
- For master failure, the computation is just aborted and it is client's responsibility to check and retry.

## Miscellaneous

- **Locality:** master attempts to schedule map tasks on or close to the machines that contains corresponding input data(input data managed by GFS is also stored in the cluster).
- **Task granularity:** ideally M and R should be large to improve load balancing and speed up failure recovery, but there are practical bounds since master needs to keep $O(M\times R)$ states in memory and each reduce task produces a separate output file.
- **Backup tasks:** when the MapReduce is close to completion, master schedules backup executions for remaining in-progress tasks to alleviate the problem of stragglers.