# Gap Analysis — 05_Databases_Storage

> Audit date: 2026-07-05. Existing files: 14. Missing (proposed): 11 (7 core + 4 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| Relational_vs_NoSQL.md | Data-model selection (relational/document/KV/column/graph survey) |
| SQL_vs_NoSQL_Scaling.md | The scaling ladder: optimize → replicate → shard → re-platform |
| Database_Normalization_Denormalization.md | 3NF, anomalies, measured denormalization |
| Database_Indexing.md | B-tree/hash/LSM index structures, write tax, EXPLAIN |
| ACID_Properties.md | Atomicity/Consistency/Isolation/Durability, WAL, MVCC, isolation anomalies |
| BASE_Philosophy.md | Basically Available, Soft state, Eventual (the AP corner) |
| CAP_Theorem.md | CP vs AP during a partition |
| PACELC_Theorem.md | Latency vs consistency when there is no partition |
| Consistency_Models.md | The consistency spectrum, quorum R+W>N |
| Replication.md | Leader-follower/multi-leader/leaderless, lag, failover |
| Sharding_Partitioning.md | Shard keys, hot spots, rebalancing |
| Distributed_Transactions.md | 2PC vs Saga, idempotency, outbox |
| Columnar_Databases.md | Column-oriented storage for OLAP |
| Data_Warehousing_Data_Lakes.md | ETL/ELT, schema-on-write vs schema-on-read, lakehouse |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **B_Tree_vs_LSM_Storage_Engines.md** — How the engine under the database actually works: page-based B-trees vs LSM memtables/SSTables, compaction strategies, read/write/space amplification, WAL mechanics and crash recovery. (Database_Indexing.md covers these as *index structures* and ACID_Properties.md uses WAL for durability, but the storage-engine treatment — "why does Cassandra write fast and read slow?" — has no home; cross-link all three.)
2. **Object_Storage_Blob_Stores.md** — S3/GCS/Azure Blob model: flat namespace, consistency guarantees, multipart upload, presigned URLs, storage classes/lifecycle tiering, and the "store the blob in object storage, the pointer in the DB" pattern every file-upload interview question needs. (Only passing S3 mentions repo-wide.)
3. **Full_Text_Search_Elasticsearch.md** — Inverted indexes, tokenization/analyzers, relevance scoring (TF-IDF/BM25), and running Elasticsearch as a *secondary datastore* kept in sync with the primary DB (CDC/dual-write pitfalls). (`12_Monitoring_Logging_Observability/Log_Management_ELK_Stack.md` uses ES only as a log sink; the search-datastore angle is absent.)
4. **Query_Optimization_Execution_Plans.md** — Cost-based optimizers, reading EXPLAIN/ANALYZE output, join algorithms (nested loop/hash/merge), statistics and cardinality estimation, N+1 queries. (Database_Indexing.md says "EXPLAIN it" but nothing teaches how to read the plan.)
5. **Database_Migrations_Schema_Evolution.md** — Expand/contract pattern, online DDL, backfills, versioned migrations, zero-downtime schema changes, schema evolution in NoSQL. (Entirely absent; a standard "how do you change a hot table" follow-up.)
6. **Time_Series_Databases.md** — Write-heavy append workloads, downsampling, retention policies, delta/gorilla compression, InfluxDB/TimescaleDB/Prometheus TSDB; the metrics/IoT interview staple.
7. **Vector_Databases.md** — Embeddings as data, ANN indexes (HNSW, IVF), recall vs latency trade-off, pgvector vs dedicated stores (Pinecone/Milvus/Weaviate). (`16_Emerging_Trends_Miscellaneous/LLM_Ops_Patterns.md` touches vector DBs inside the RAG flow, but the datastore itself — increasingly a first-class interview topic — has no dedicated file; cross-link the two.)

### Optional / nice-to-have

8. **Transaction_Isolation_Levels_MVCC.md** — Read committed → repeatable read → serializable, snapshot isolation, write skew, SSI. (Substantially covered inside ACID_Properties.md §4 "Isolation Levels & Read Anomalies"; split out only if that file stays a four-properties overview.)
9. **Bloom_Filters_Probabilistic_Data_Structures.md** — Bloom/cuckoo filters (LSM read paths, cache-miss avoidance), HyperLogLog, count-min sketch. (Only passing mentions in Database_Indexing.md.)
10. **Graph_Databases.md** — Property graphs, Cypher/Gremlin, index-free adjacency vs join tables, when a graph DB beats sharded relational for traversals. (Surveyed in one row of Relational_vs_NoSQL.md.)
11. **NewSQL_Distributed_SQL.md** — Spanner/CockroachDB/TiDB, TrueTime and clock-bound commits, how they square CAP. (Named as the final "re-platform" rung in SQL_vs_NoSQL_Scaling.md but never explained.)

## Deliberately NOT flagged (covered in other folders)

- Database caching layers, cache-aside/read-through, eviction → `06_Caching_Performance/Caching_Strategies.md`, `Cache_Eviction_Policies.md`
- Connection pooling → `06_Caching_Performance/Database_Connection_Pooling.md` (duplicated in 14 — see housekeeping there)
- Consistent hashing (shard/replica placement) → `07_Load_Balancing_Scalability/Consistent_Hashing.md`
- Change Data Capture / outbox relay mechanics → `09_Message_Queues_Event_Streaming/Change_Data_Capture_CDC.md`
- Materialized views / precomputation → `14_Performance_Optimization_Scalability/Precompute_Materialized_Views.md`
- Optimistic vs pessimistic locking → `14_Performance_Optimization_Scalability/Optimistic_vs_Pessimistic_Locking.md`
- Backups, disaster recovery, RPO/RTO → `13_Cloud_Native_Infrastructure/Disaster_Recovery_RPO_RTO.md`
- Consensus (Raft/Paxos) behind replication failover → `10_Microservices_Distributed_Systems/Distributed_Consensus_Raft_Paxos.md`

## Folder housekeeping

- **CAP/PACELC triple coverage** — this folder has `CAP_Theorem.md` + `PACELC_Theorem.md` while `01_Planning_Foundations` has `CAP_Theorem_PACELC.md` covering the same two theorems. Pick one canonical home (the deep dives here fit the section spine) and reduce the 01 file to a planning-level summary that links here — or merge; three files for two theorems will drift.
- **`06_Caching_Performance/Database_Connection_Pooling.md` and `14_Performance_Optimization_Scalability/Database_Connection_Pooling.md` are duplicate filenames** for the same topic in two folders; adjacent to this section's scope, worth deduplicating when either folder is next touched.
- **README index will need extending** — the README's mermaid map, 14-row file index, study paths, and Decision Card are hard-coded to the current topics; the new files slot naturally into MODEL (graph/vector/time-series/object storage), INDEX (storage engines, query plans), and a new "operate & evolve" cluster (migrations).
