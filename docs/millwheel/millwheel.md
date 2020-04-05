# MillWheel: Fault-Tolerant Stream Processing at Internet Scale

*A framework for building streaming systems with fault tolerance, persistent state and scalability.*

## Background

- Other streaming systems do not provide the combination of fault tolerance, versatility and scalability.
- A monotonically increasing low watermark, which indicates all data up to a given timestamp has been received, is useful to distinguish whether there is no data or data is delayed.

## System Overview

- MillWheel is a graph of user-define transformations on input data that produces output data.
- Inputs/outputs are represented by (key, value, timestamp) triples.
- User code can access a per-key, per-computation persistent state for per-key aggregation.
- **Delivery Guarantee:** all internal updates within the MillWheel framework resulting from record processing are atomically checkpointed per-key and records are delivered exactly-once.

## Concepts

- A computation subscribes to zero or more input streams and publishes one or more output streams.
- Persistent states are per-key opaque byte strings stored in Bigtable/Spanner.
- Consumer specifies per-stream key extraction functions.
- Computation code(containing application logic) is invoked upon receipt of input data. Processing is serialized per-key but can be parallelized over distinct keys.
- Low watermark of a computation A is defined as min(oldest work of A, low watermark of C: C outputs to A)​. Low watermark values are seeded by injectors that send data into MillWheel from external systems.

![system](images/system.jpg)

## Fault Tolerance

### Delivery Guarantee

- System assigns unique IDs to all records at production time. Upon receipt of an input record, MillWheel checks the record against deduplication data(bloom filter + backing store).
- For strong productions, MillWheel checkpoints produced records before delivery in the same atomic write as state modification. When a process restarts, the checkpoints are scanned into memory and replayed.
- For weak productions, MillWheel broadcasts downstream deliveries optimistically prior to persisting state. MillWheel selectively checkpoints a small percentage of straggler productions to prevent them from occupying undue resources in the sender.

![weak production](images/weak-production.jpg)

### State Manipulation

- To avoid inconsistencies in persistent state, all per-key updates are wrapped in a single atomic operation.
- To address zombie writers and stale writes in network, MillWheel guarantees that for a given key, only a single worker can write to that key at a particular point in time. It is implemented by attaching a sequencer token to each write and let backing store check for validity.
- Single-writer guarantee is critical to state consistency and low watermark correctness.

## System Implementation

- A replicated master divides each computation into a set of owned lexicographical key intervals and assigns intervals to a set of machines. It can move/split/merge intervals.
- Each interval is assigned a unique sequencer and is invalidated whenever the interval is changed(for single-writer guarantee).
- Low watermarks are tracked by a central authority. Workers compute low watermark updates and report to the central authority, so the central authority's low watermark values are always as conservative as workers'.

