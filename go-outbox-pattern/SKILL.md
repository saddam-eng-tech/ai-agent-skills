name: go-outbox-pattern
description: Scaffolds the transactional outbox pattern for a Go microservice — SQL migration for the outbox table, transactional domain write + outbox insert in one DB transaction, a polling worker goroutine, and a Kafka publisher with consumer idempotency check.

# Go Outbox Pattern

Generates the full transactional outbox pattern: write to domain table AND outbox table in one transaction, then a background worker reliably publishes outbox rows to Kafka — so no events are ever lost even on crash.

**Input**: Entity name, Kafka topic name, event types (Created/Updated/Deleted), module path
**Output**: Outbox migration, domain+outbox transactional write, polling worker, Kafka publisher, consumer idempotency table

## Trigger phrases
- "outbox pattern"
- "dual write problem"
- "DB and Kafka consistency"
- "transactional messaging"
- "reliable event publishing"
- "avoid losing events on crash"
- "how do I publish to Kafka and write to Postgres atomically"

## Output files
- `migrations/000002_create_outbox.sql`
- `migrations/000003_create_processed_events.sql`
- `internal/outbox/outbox.go`
- `internal/outbox/postgres_repo.go`
- `internal/outbox/worker.go`
- `internal/outbox/kafka_publisher.go`

## Source
`SKILL-8.md`