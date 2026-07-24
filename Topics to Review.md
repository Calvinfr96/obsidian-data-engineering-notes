Looking across all of your stories, I see recurring themes.

## Distributed systems

You mention:

- caches
- race conditions
- consistency
- retries
- synchronization
- deployments

I'd want to be comfortable explaining:

- eventual consistency
- strong consistency
- why race conditions occur
- optimistic vs pessimistic locking
- idempotency
- retries with backoff
- circuit breakers

---

## Data modeling

Several stories involve redesigning schemas or changing how data is stored.

I'd review:

- normalization vs denormalization
- partition keys
- hot partitions
- schema evolution
- serialization
- Protobuf
- JSON
- Parquet
- Avro

---

## AWS

Since virtually every story references AWS services, I'd review the services you actually mention:

- DynamoDB
- ElastiCache
- Redis
- CloudWatch
- IAM
- KMS
- Lambda
- SNS
- SQS

Not just what they are—but **why you used them instead of something else**.

---

## Concurrency

This comes up repeatedly.

I'd review:

- threads
- async processing
- locks
- race conditions
- deadlocks
- queues
- producer/consumer patterns

---

## Performance

You mention:

- p99 latency
- CPU
- memory
- cache hit ratio

I'd understand:

- why p99 matters more than averages
- cache hit ratio
- serialization overhead
- network latency
- memory vs CPU tradeoffs