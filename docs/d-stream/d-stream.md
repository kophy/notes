# Discretized Streams: Fault-Tolerant Streaming Computation at Scale

*A new stream processing model with efficient fault recovery and straggler mitigation.*

## Background

- There is a need for streaming computation models that can scale transparently to large clusters, where faults and stragglers are inevitable.
- Most existing systems are based on continuous operator model(like Storm). The stateful nature of operators and nondeterminism of records make it hard to provide efficient fault recovery.
  - Two main approaches, replication(2x hardware) or upstream backup(long replay time), are both expensive.
  - Neither approach handles stragglers -- in replication, synchronization protocols slow both down replicas; in upstream backup, a straggler must be treated as a failure. 
- We seek a system design with scalability to hundreds of nodes, minimal cost, second-scale latency and second-scale fault/straggler recovery.

## Discretized Streams

- D-Stream model treats a streaming computation as a series of deterministic batch computations on small time intervals.
- Users define programs by manipulating D-Stream objects; a D-Stream is a sequence of RDDs.
- The consistency semantics are clear because time is naturally discretized into intervals and each interval's output RDDs reflect all inputs received up to that interval.
- D-Streams place records into input datasets based on the time when each records *arrives* at the system. Grouping records by external timestamps is not natively supported.

![overview](images/overview.jpg)

## System Architecture

- Spark Streaming consists of three components, (1) a master to track D-Stream lineage and schedules tasks, (2) worker nodes to receive/store/process data and computed RDDs and (3) a client library to send data into system.
- To decide when to process a new interval, nodes synchronize clocks via NTP and send the master a list of received block IDs in each interval when it ends. The master then starts launching tasks for that interval without further synchronization.
- Spark Streaming divides computations into short, stateless, deterministic tasks, each of which may run on any node in the cluster. Moving computations to another machine is hard in traditional streaming systems with rigid topologies.
- To support streaming, Spark Streaming implements changes/optimizations like timestep pipelining, lineage cutoff, master recovery, etc.

![components](images/components.jpg)

## Fault and Straggler Recovery

- **Parallel recovery:** when a node fails, the state RDD partitions and running tasks on that node can be recomputed in parallel on other nodes.
- **Straggler mitigation:** run speculative backup copies of slow tasks like in batch systems.
- **Master recovery:** workers can simply connect to a new master since there is no problem if a given RDD is computed twice(it is fine to lose some running tasks).