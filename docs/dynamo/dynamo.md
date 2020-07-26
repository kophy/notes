# Dynamo: Amazon’s Highly Available Key-value Store

*A highly available key-value storage system for "always-on" experience.*

## Background

- The Amazon platform is built on top of tens of thousands of server and network components, where there are always a small but significant number of components failing at any given time.
- Some applications like shopping cart need always-available storage technologies for customer experience.

## Introduction

- Dynamo provides only simple key-value interface; no operations span multiple data items.
- Unlike traditional commercial systems putting importance on consistency, Dynamo sacrifices consistency under certain failure scenarios for availability.
- Dynamo uses a synthesis of well known techniques to achieve scalability and availability:

![summary](images/summary.png)

## System Architecture

- **Consistent hashing:** To scale incrementally, Dynamo uses consistent hashing to partition data. The principle advantage is that arrival/departure of a node only affects immediate neighbors.
- **Virtual node**: Dynamo introduces the concept of virtual nodes to balance the load when membership changes and account for heterogeneity in the physical infrastructure (virtual nodes on same physical node are skipped in replication).
- **Data Versioning**: Dynamo uses vector clocks(list of <node, counter\> pairs) to capture the causality between different  versions of the same object. If Dynamo can't resolve divergent versions, it will return all objects and let applications resolve the conflicts.
- **Sloppy quorum and hinted handoff:** Dynamo uses quorum-based consistency protocol(R+W>N),  but does not ensure strict quorum membership. When a node A is temporarily unavailable, another node B will help maintain the replica and deliver the replica to A when detecting A has recovered.
- **Replica synchronization:** Dynamo anti-entropy protocol uses hash trees to reduce the amount of data need to be transferred to detect replica inconsistencies.
- **Gossip protocol:** Each node contacts a random peer every second and two nodes reconcile their persisted membership change histories and views of failure state. To prevent logical partitions, some seed nodes are known to all nodes.