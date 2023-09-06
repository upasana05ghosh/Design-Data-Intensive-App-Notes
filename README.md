- [Design-Data-Intensive-App-Notes](#design-data-intensive-app-notes)
- [Chapter 1: Reliable, Scalable \& Maintainable](#chapter-1-reliable-scalable--maintainable)
    - [Reliable](#reliable)
    - [Scalability](#scalability)
    - [Maintainability](#maintainability)
- [Chapter 2: Data Models and Query Languages](#chapter-2-data-models-and-query-languages)
    - [Advantage](#advantage)
    - [Disadvantage](#disadvantage)
    - [Conclusion](#conclusion)
- [Chapter 3: Storage and Retrieval](#chapter-3-storage-and-retrieval)
    - [Families of Storage Engine](#families-of-storage-engine)
    - [Data structure that power db](#data-structure-that-power-db)
    - [SSTable](#sstable)
    - [B Tree](#b-tree)
- [Chapter 4: Encoding \& Evolution](#chapter-4-encoding--evolution)
    - [What is encoding](#what-is-encoding)
    - [Why encode?](#why-encode)
    - [Tools available for encoding](#tools-available-for-encoding)
    - [Data flow](#data-flow)
- [Chapter - 5: Replication](#chapter---5-replication)
    - [Replication](#replication)
    - [Single Leader](#single-leader)
      - [Sync Vs Async Replication](#sync-vs-async-replication)
      - [Handling node outage](#handling-node-outage)
      - [Implementation of Replication Log](#implementation-of-replication-log)
      - [Problem with Replication Log](#problem-with-replication-log)
    - [Multi-Leader Replica](#multi-leader-replica)
    - [Leaderless Replication](#leaderless-replication)
- [Chapter 6: Partitioning](#chapter-6-partitioning)
    - [What is Partitioning?](#what-is-partitioning)
    - [Why Partition?](#why-partition)
    - [Partition \& Replication](#partition--replication)
    - [Partition of Key-value data](#partition-of-key-value-data)
      - [Approach to Partition](#approach-to-partition)
      - [Skewed workload and relieving hot spot](#skewed-workload-and-relieving-hot-spot)
    - [Partitioning and Secondary Index](#partitioning-and-secondary-index)
    - [Rebalancing Partitions](#rebalancing-partitions)
    - [Request Routing](#request-routing)


# Design-Data-Intensive-App-Notes
Design Data Intensive App Notes

# Chapter 1: Reliable, Scalable & Maintainable

### Reliable
 * App performs as expected
 * Can tolerate user mistake
 * Performance is good
 * System prevents unauthorize access and abuse

### Scalability
  * Ability to cope with increase load
  * Describe load
      * DB -> Ratio of R/w
      * Server -> Request/sec
      * Cache -> Hit rate
      * Online server -> response time
           * P50 -> half of user receives request
       
### Maintainability
  * Operability -> good monitor and documentation
  * Simplicity -> easy to understand
  * Evolvability -> Easy to make new changes

---
# Chapter 2: Data Models and Query Languages

| SQL | NOSQL   | 
| :---:   | :---: |
| Relational | JSON |

### Advantage
| SQL | NOSQL   | 
| :---:   | :---: |
| Query optimiser is available | Schema flexibility |
| Better support for n:1, n:m relation | Better performance due to locality|
|  |Close to data structure for mapping|

### Disadvantage
| SQL | NOSQL   | 
| :---:   | :---: |
| Schema change is long and slow | Weak support for joins |
| Better support for n:1, n:m relation | Difficult with n:m relation|
|  |On Update, entire document update|

### Conclusion
Go with hybrid

---
# Chapter 3: Storage and Retrieval
### Families of Storage Engine
   * Log structured
      * LSM tree
   * Page oriented
     * B tree

### Data structure that power db
  * db_set(k,v)
     * append value at the end
     * O(1)
  * db_get(k)
    * scan all entries to get the most recent value
    * O(n)
  * Issue?
     * For get, we need to scan entire entry
   * Solution?
      * Use Index
   * Another Issue -> we may run out of disk since we are appending values
   * Solution?
      * Break logs into small segments
      * Perform compaction on segments

  ### SSTable
   *  sequence of key-value pair is sorted by key
   * known as Sorted String Table (SSTable)
   * Why SSTables?
       * Merging segment is simple and efficient
       * No need to index all keys, keep an sparse index
   * How to create SSTables?
       * In Memory → AVL Tree/ Red Black tree
          * Adv → insert key in any order, Read them back in sorted order
       * Write -> add it to in-memory balanced tree data str. (called memtable)
          * When it become bigger → write it to disk as an SStable file
       * Read -> first try to find key in memtable, then in most recent on-disk segment then next segment on disk
       * Background → run merging and compaction to combine segment files. 
   * Issue? 
       * If a DB crashes, most recent writes in memtable → lost
   * Solution? 
       * Keep a separate log on disk. Every write is immediately appended
       * Every time memtable is written out of SSTable, log is discarded

   ### B Tree
   | SSTable | B-Tree |
| --- | --- |
| Variable size segments | Fixed size block/page, more close to disk (4KB in size) |
| In-memory and then in disk | Disk |
| write → append only | Write → overwrite & risk of Db failure at the time of overwrite |
| Faster for write | Faster for read |

 ---
# Chapter 4: Encoding & Evolution
### What is encoding
  * Convert data from in-memory to byte sequence

### Why encode?
 * Send data over network
 * Write to a file

### Tools available for encoding
  * Lang specific
     * Ex - java - java.io.seralization
     * Pros - easy to use
     * Cons - programming lang specific,
         * not forward/backward compatible
   * JSON, XML
     * Issue - can't distinguish between number and string
       * Don't support binary string
   * Binary Encoding
     * Thrift (FB), ProtoBuff (Google), Avro
       - Open source
       - Required a schema for encoding
       - Mostly backward and forward compatible

 ### Data flow
   * Via DB
   * via service call
     - Rest, Soap, RPC(gRPC)
   * via Async message passing
     - Kafka

 ----
 # Chapter - 5: Replication
 
### Replication
* What - Keeping a copy of same data on multiple machines that are connected via network.
* Why?
   - Reduce access latency
   - Increase availability
   - Increase read throughput
 * Algorithm for replicating changes between nodes
   - Single leader
   - Multi leader
   - Leaderless
  
 ### Single Leader
 - called Leader based replication/ master-slave replication
 - Write -> only accepted on leader
 - Read -> leader or any follower

#### Sync Vs Async Replication
- Sync
   - Adv -> followers always up-to-date
   - Disadv -> if followers doesn't respond, write won't proceed. It's impractical for all followers to be in sync.
   - Solution -> One of the follower to be in sync. Called 1 sync follower or semi sync.

 #### Handling node outage
 - Follower failure -> catch up recovery
 - Leader failure -> failover

#### Implementation of Replication Log
- Statement based replication -> every statement is forwarded to followers
- WAL (write ahead log) -> Log send to follower
- Logical(row-based) log replication -> only those values are send which are updated/inserted/deleted and not the entire row.
- Trigger based replication

#### Problem with Replication Log
- Reading your own write -> User submitted data and want to view it immediately. 
- Monotonic Read -> User reads from several replicas, they may see things moving backward in time. 
- Consistent Prefix Read -> If sequence of write happens in a order, read should see them in order.

### Multi-Leader Replica
- When
   - Multi-data-center operation -> leader in each center
   - Client with offline operation -> calendar
   - Collaborative editing -> google docs
 
 - Handling write conflicts
    - Conflict avoidance -> all write for a particular record goes through same leader
    - Converging towards a consistent state.
  
  ### Leaderless Replication
  - Quorums for reading and writing
    - If n replicas, every write must be confirmed by w nodes and at least r nodes for each read
    - w + r > n
      
 ---
 # Chapter 6: Partitioning

### What is Partitioning?
- Way of breaking a large dataset into smaller ones.

### Why Partition?
- To spread the data and the query load evenly across nodes.

### Partition & Replication
- Partition is usually combined with replication so that copies of each partition are stored on multiple nodes for fault tolerant.

### Partition of Key-value data
- Access record by primary key
- Skewed -> If partition is unfair, some partition will have more data than other. 
- Hot spot -> A partition with disappropriately high record. 

#### Approach to Partition
* Partition by Key Range
  * Assign a continuous range of keys to each partition. Keys will be sorted
  * Adv - Range queries will be efficiently searched.
  * Issue - Risk of hot spot
  * Solution - Partition boundaries need to adapt to data

* Partition by Hash of key
  * use hash function to determine the partition of a given key.
  * Assign each partition a range of hashes (rather than a range of keys)
  * Issue -> Ability to do efficient range queries
  * Solution -> For range queries, search in all partition and then combine the result.
  
* Hybrid approach
  * compound key
  * first part of the key -> to identify the partition and
  * other part -> for the sort order
  * Ex -> social media site, one user may post many updates -> (user_id, update_ts)

#### Skewed workload and relieving hot spot
* Hashing a key -> can help reduce hot spot
* Few cases where a key become very hot:
  * Sol -> Add a random number to the beg or end of the key
  * Issue -> Reading will become more expensive and we need additional book keeping

### Partitioning and Secondary Index
* Secondary index -> doesn't identify a record uniquely but rather is a way of searching for occurrences of a particular value
  
* Document-partitioned Indexes (local index)
  * Each partition maintain it's own secondary index.
  * Adv - Write to a document -> you only need to update the index in that document.
  * Issue - For reading, we need to send request to all the partition, combine all result back. 
  
* Term-partitioned index (global indexes)
  * Construct index that cover data in all partition.
  * This global index must be partitioned too.
  * Adv -> Make read more efficient
  * Disadv -> write will be slow and complicated. Write to single doc will affect multiple partition of the index. 

### Rebalancing Partitions
* Why Rebalance -> things changes in DB.
  * Dataset size increases -> add more disk/ram
  * a machine fails and other machine need to take over the failed node
  * Read throughput increases and so need more CPUs.

* Strategies for Rebalancing
  * How not to do: hash mod N
    * Issue -> the no. of nodes N changes
  
  * Fixed number of partition
    * Create many more no. of partition than there are nodes. 
    * If 10 nodes, 1000 partition and each node will have 100 partition. 
    * New node added -> steal a few partition from each existing nodes.
    * Issue -> choosing the right number of partition is difficult if size of dataset is highly variable.

    * Dynamic Partition
      * Used by key range partition DB
        * when a partition grows to exceed a configured size, split it into two partitions.
        * When a partition shrinks below some threshold, merge with adjacent. 

   * Partition proportionally to nodes
     * Fixed no. of partition per node
     * New node add -> randomly choose fixed no. of existing partition to split.

### Request Routing
* When a client make a request, how does it know which node to connect to?
  * Service discovery
  * Zookeeper -> keep track of the cluster metadata



 
    
