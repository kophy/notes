# FlumeJava: Easy, Efficient Data-Parallel Pipelines

*The combination of high-level abstractions for parallel data and computation, deferred evaluation and optimization, and efficient parallel primitives yields an easy-to-use system that approaches the efficiency of hand-optimized pipelines.*

## VS MapReduce

- Real-life computations require a chain of MapReduce stages. FlumeJava can optimize the execution plan and choose implementation strategy(local loop, remote MapReduce, etc) when running the execution plan.
- FlumeJava is easier to develop and test. Parallel collections abstract away the details of data representation and parallel operations abstract away the implementation strategy.
- FlumeJava automatically deletes temporary intermediate files when no longer needed.

## Basics

- Data:
  - immutable bag of elements `PCollection<T>`.
  - immutable multi-map `PTable<K, V>`.
- Operations:
  - `parallelDo` for map/reduce.
  - `groupByKey` for shuffle.
  - `combineValues` is a special case of `parallelDo`. It is more efficient since MapReduce combiner is allowed.
  - `flatten` views a list of `PCollection<T>` as a single `PCollection<T>`(no copy).
  - `join` returns `PTable<K, Tuple<Collection<V1>, Collection<V2>>>` and is implemented with intermediate type `PTable<K, TaggedUnion2<V1, V2>>`.

## Optimizer

### ParallelDo Fusion

- Producer-consumer fusion: replace $f(g(x))$ with $(g + f \circ g)(x)$.
- Sibling fusion: replace $f(x) + g(x)$ with $(f+g)(x)$.

### MSCR Fusion

MSCR(MapShuffleCombineReduce) operation is the intermediate operation to help bridge the gap between(1) combinations of operations and (2) single MapReduces.

![MSCR](images/MSCR.jpg)

### Strategy

The optimizers performs multiple passes over the execution plan to produce the fewest, most efficient MSCR operations.

1. Sink Flattens: create opportunities for ParallelDo fusion.
2. Lift CombineValues operations: CombineValues immediately follows GroupByKey is subject to ParallelDo fusion.
3. Insert fusion blocks: for ParallelDos between two GroupByKeys, FlumeJava needs to estimate size of intermediate output and mark boundary to block ParallelDo fusion.
4. Fuse ParallelDos.
5. Fuse MSCRs.

## Executor

Batch execution: FlumeJava traverses the operations in the execution plan in forward topological order.Independent operations are operated simultaneously.