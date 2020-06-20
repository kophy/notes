# On Brewing Fresh Espresso: LinkedIn’s Distributed Data Serving Platform

*A timeline-consistent document-oriented distributed database to address Linkedin's requirements for primary store.*

## Background

- **Timeline Consistency**: events are applied in the same order on all replicas.
- Linkedin used to have a single RDBMS with user data tables; two derived data systems, for full-text search and relationship graph traversal, are kept up-to-date by Databus.
- RDBMS has pain points like scaling and schema management, while most Linkedin primary data does not require full RDBMS functionality.
- Voldemort, a Dynamo-like data store, is increasingly being used for primary data, but key-value model does not support secondary indexing very efficiently.

## Data Model

- Linkedin mainly deals with two forms of relationships, nested entities and independent entities. Applications need atomicity constraints for updating nested entities, but not for independent entities.
- Espresso uses a hierarchical data model to model nested entities efficiently. Independent entities are modeled as disjoint entities with change capture stream to materialize relationships.
- The data hierarchy is composed of document(smallest data unit with primary key), table(collection of like-schema-ed documents), document group(collection of documents with common partition key) and database.
- All documents within a database are partitioned with the same partitioning strategy.
- Document group is a logical concept and can span across tables; it is the largest unit with transactionality support(same partition).

## System Architecture

- Espresso is composed of: clients and routers, storage nodes, relays(Databus) and cluster manager(Helix). 
- Storage nodes maintain both base data and local secondary indexes. To achieve read-after-write consistency, updates are applied transactionality to base data and local secondary indexes.
- Data is over-partitioned to reduce load during cluster expansion. For each partition, Helix assigns one storage node as master and rest as slaves. Committed changes are pulled into Databus via local transaction log and slave replicas consume change streams from Databus.

![architecture](images/architecture.png)

## (Local) Secondary Index

- Local secondary indexes(for document groups) must be updated in real time, while global secondary indexes(for independent entities) can be updated asynchronously.
- The first approach is based on Lucene. We create per-collection small indexes to reduce memory footprint and store them in MySQL to achieve transactionality.
- The second approach is called Prefix Index. We prefix each term in the index with the collection key(equivalent to having per-collection small indexes); terms are store on a B+ tree structure and corresponding document lists are stored in MySQL. This avoids the overhead of frequently opening/closing indexes and re-indexing the whole document to update indexes.