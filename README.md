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
   - Increae read throughput
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
   - Disadv -> if followers doesn't respond, write won't proceed. It's impracticle for all followers to be in sync.
   - Solution -> One of the follower to be in sync. Called 1 sync follower or semi sync.

 #### Handling node outage
 - Follower failure -> catch up recovery
 - Leader faillure -> failover

#### Implementation of Replication Log
- Statement based replication -> every statement is forwareded to followers
- WAL (write ahead log) -> Log send to follower
- Logical(row-based) log replication -> only those values are send which are updated/inserted/deleted and not the entire row.
- Trigger based replication

#### Problem with Replication Log
- Reading your own write -> User submitted data and want to view it immediately. 
- Monotonic Read -> User reads from several replicas, they may see things moving backward in time. 
- Conistent Prefix Read -> If sequence of write happens in a order, read should see them in order.

### Multi-Leader Replica
- When
   - Multi-datacenter operation -> leader in each center
   - Client with offline operation -> calendar
   - Collaborative editing -> google docs
 
 - Handling write conflicts
    - Conflict avoidance -> all write for a particular record goes through same leader
    - Converging towards a consistent state.
  
  ### Leaderless Replication
  - Quorums for reading and writing
    - If n replicas, every write must be confirmed by w nodes and at least r nodes for each read
    - w + r > n
      
 
 
    
