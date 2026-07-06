# Gap Analysis — 09_Message_Queues_Event_Streaming

> Audit date: 2026-07-05. Existing files: 10. Missing (proposed): 6 (3 core + 3 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| Kafka_Architecture.md | Brokers, topics, partitions, sequential-log internals |
| RabbitMQ_vs_Kafka.md | Smart-broker routing vs durable replayable log |
| Consumer_Groups_Rebalancing.md | Horizontal consumption, partition assignment, rebalancing |
| Message_Ordering_Offsets.md | Per-partition ordering, partition keys, offset management |
| Exactly_once_Semantics.md | Delivery guarantees (incl. at-most/at-least-once framing), idempotent producers, transactions |
| Dead_Letter_Queues_Retry_Strategies.md | Poison messages, backoff + jitter, DLQ isolation |
| Backpressure_Throttling.md | Pull-based flow control, token/leaky bucket, load shedding |
| Event_Sourcing_Pattern.md | Event log as source of truth, replay, snapshots |
| Change_Data_Capture_CDC.md | Binlog/WAL tailing, Debezium, avoiding dual writes |
| Stream_Processing.md | Kafka Streams/Flink basics, windowing (tumbling/hopping/session) |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **Schema_Registry_Message_Schema_Evolution.md** — Confluent Schema Registry, backward/forward/full compatibility modes, evolving producer/consumer contracts without breaking downstream, subject naming strategies. (Wire formats themselves live in `14_Performance_Optimization_Scalability/Binary_Protocols_Protobuf_Avro.md`; the registry + evolution governance story has no home and "how do you change an event's schema safely?" is a standard follow-up.)
2. **Log_Compaction_Retention_Policies.md** — Queue vs stream vs compacted-log semantics, time/size retention vs compaction, compacted topics as changelogs, tombstones, the stream–table duality. (Log compaction has zero mentions anywhere in the repo despite being a classic Kafka probe.)
3. **Delayed_Messages_Priority_Queues.md** — Scheduled/delayed delivery (SQS delay, RabbitMQ TTL + dead-letter-exchange trick, timing wheels), priority queues and starvation, why Kafka has neither natively and the workarounds. (Comes up in "design a job scheduler / reminder system"; no dedicated file exists.)

### Optional / nice-to-have

4. **Kafka_vs_Pulsar_vs_Kinesis.md** — Pulsar's segment-based storage (BookKeeper) and tiered storage, Kinesis shards vs Kafka partitions, managed vs self-hosted brokers. (Pulsar/Kinesis appear only as two passing mentions in Kafka_Architecture.md.)
5. **Kafka_Streams_vs_Flink.md** — Stateful stream processing depth: state stores, checkpointing/exactly-once in Flink, watermarks and late events, library vs cluster deployment model. (Stream_Processing.md covers windowing basics — this is the senior-level extension; cross-link rather than duplicate.)
6. **Fan_out_on_Write_vs_Read.md** — Push vs pull feed fan-out, hybrid celebrity handling, fan-out worker queues. (The classic "design Twitter/newsfeed" pattern; pub/sub mechanics are in `08_Communication_Protocols/Message_Queues_PubSub.md` but the fan-out trade-off itself has no dedicated file.)

## Deliberately NOT flagged (covered in other folders)

- Pub/sub fundamentals, exchange/routing topologies → `08_Communication_Protocols/Message_Queues_PubSub.md`
- Event-driven architecture (the "why" above the brokers) → `03_High_Level_Design/Event_Driven_Architecture.md`
- Transactional Outbox/Inbox, Saga, CQRS → proposed in `10_Microservices_Distributed_Systems/GAPS.md` (planned there; cross-link once written)
- Binary serialization formats (Protobuf/Avro encoding itself) → `14_Performance_Optimization_Scalability/Binary_Protocols_Protobuf_Avro.md`
- API-level rate limiting (vs consumer-side throttling here) → `02_API_Interface_Design/Rate_Limiting.md`
- Batch vs stream processing trade-off → `14_Performance_Optimization_Scalability/Batching_vs_Streaming.md`

## Folder housekeeping

- No structural issues found: README.md exists, indexes all 10 files, and matches the section conventions.
- When gap files are added, update README's mermaid map, file index, and study paths (they currently hard-code "the 10 topics").
