# Goal
This document lists my personal notes taken while reading the [DynamoDB research paper](http://www.allthingsdistributed.com/2007/10/amazons_dynamo.html).

## Notes

- DynamoDB is a highly available key-value storage system.
- High Availability is achieved by sacrificing consistency under certain failure scenarios.
- Object versioning is used.
- Application assisted conflict resolution is used.
- DynamoDB has to be performant, reliable, efficient and scalable.
- Reliability is the most important factor as it affects customer trust and has financial consequences.
- As failure is very common at Amazon's datacenters, systems need to be designed so that failure doesn't affect performance or availability.
- Reliability and scalability of a system is dependent on how its application state is managed.
- Dynamo is used to manage the state of services that have very high reliability requirements and need tight control over the tradeoffs between availability, consistency, cost-effectiveness and performance.
- Dynamo provides a simple primary-key only interface to meet the requirements of applications that only need primary-key access to datastore. Such applications include services that provide best seller lists, shopping carts, customer preferences, session managementm sales rank and product cataog.
- Data is *partitioned* and replicated using *consistent hashing*.
- Consistency is facilitated by **object versioning**.
- Consistency among replicas during updates is maintained by a **quorum**-like technique and **decentralized replica synchronization protocol**.
- A **Gossip** protocol is used for failure detection and maintaining membership list.
- Minimal need of manual intervention for partitioning and redistribution.
- The internal Amazon services that used Dynamo as its underlying storage technology, were able to scale to extreme peak loads efficiently without any downtime (eg. Shopping Cart Service).
- What Dynamo demonstrates is the possibility of using an **eventually-consistent** storage system in production with demanding applications.
- Dynamo has a simple key/value interface, is highly available with a clearly defined consistency window, is efficient with its resource usage and has  a simple scale out scheme to address growth in data set size or request rates.
- Each service that uses Dynamo runs its own Dynamo instances.

## The requirements that Dynamo users (services) share
### Query Model
- Read and Write operations to a data item uniquely identified by a key.
- State is stored as binary objects (blobs) identified by unique keys.
- No operations span multiple data items.
- No need for relational schema.
- The objects that are typically stored within Dynamo are relatively small (< 1MB).

### ACID Properties (Atomicity, Consistency, Isolation, Durability)
- Dynamo targets applications taht operate with weaker consistency if this results in high availability.
- Dynamo does not provide any isolation guarantees and permits only single key updates.

### Efficiency
- The system needs to function on a commodity hardware infrastructure.
- Services are Amazon have stringent latency requirements measured at 99.9th percentile of the distribution.
- As state access plays a crucial role in service operation, the storage system must be capable of meeting such stringent SLAs.
- Services must be able to configure Dynamo such that they consistently achieve their latency and throughput requirements. The tradeoffs are in performance, cost efficiency, availability and durability guarantees.

### Security
- No need for authentication and authorization as the Dynamo is only used by Amazon's internal services.

### Scalability
- As each srvice uses its distinct instance of Dynamo, its initial design targets a scale up of up to hundreds of storage hosts.

## Dynamo Design Principles
### Incremental Scalability
- Dynamo should be able to scale out one storage host (node) at a time, with minimal impact on both operators of the system and the system itself.

### Symmetry
- There are no special peers. This facilitates the process of system provisionning and maintenance.

### Decentralization
- Favorising decentralized P2P techniques over centralized control. This lead to a simpler, more scalable, and more available system.

### Heterogeneity
- The ability to run on a heterogeneous park of peers (with different resource capabilities).

## About consistency vs. availability
- Dynamo is designed to be an eventually consistent data store; that is all updates reach all replicas eventually.
- For systems prone to server and network failures, availability can be increased by using optimistic replication techniques, where changes are allowed to propagate in to replicas int he background, and concurrent, disconnected work is tolerated.
- Conflicting changes must be detected and resolved.
- The question is: **When to resolve conflicts** and **Who is responsible for doing it**.
- When to resolve conflicts: on Read? on Write? 
- Dynamo targets the design space of an *always writeable* data store (see Services section below). So conflict resolution is done at the read level.
- Who resolves conflcits? Either at the data store level or at the application level. If it's done at the data store level, the conflict resolution policies can only be simple like: *last write wins*. The advantage of doing it at the application level is to let it decide which policy to choose.

## Services at Amazon
- **Stateless service** is a service which aggregates responses from other services.
- **Stateful service** is a service that generates its reponse by executing business logic on its state storeted in persistent store.
- Example of services: Shopping Cart, Fulfillment, Recommendations, Fraud Detection.
- Services are defined through a well defined interface and are accessible over the network.
- A significant amount of services don't need any relational schema.
- For a number of Amazon services, rejecting customer updates could result in a poor customer experience. For instance, the Shopping Cart Service must allow customers to add and remove items from their shopping cart even amidst network and server failures. This is why the complexity of conflict resolution is pushed to the reads, in order to ensure that writes are never rejected.

### SLA at Amazon (Service Level Agreement)
- While industry standard describes a performance oriented SLA using average, median and expected variance, Amazon finds that these metrics are not good enough if the goal is to build a system where all customers have a good experience.
- Instead, Amazon expresses and measures SLAs at the 99.9th percentile of the distribution.
- Techniques such as the load balanced selection of write coordinators, are purely targeted at controlling performance at the 99th percentile.
- State management (and therefore Dynamo) is the main component of a service's SLA since their business logic is lightweight.
- One of the main design considerations for Dynamo is to give services control over their system properties, such as durability and consistency and to let services make their own tradeoffs between functionality, performance and cost effectiveness.

## Questions
- The Shopping Cart Service uses Dynamo which sacrifices consistency for availability. How is that possible? I would have imagined that a shopping cart requires strong consistency?
