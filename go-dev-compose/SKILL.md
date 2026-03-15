name: go-dev-compose
description: Generates a production-quality docker-compose.yml for local development of Go microservices — with health checks, correct dependency ordering, named volumes, environment files, and a Makefile with dev/dev-down/logs targets.

# Go Dev docker-compose Generator

Generates a well-structured `docker-compose.yml` with health checks, correct startup ordering, shared infrastructure (Postgres, Redis, Kafka), and a `make dev` workflow for spinning up the full local stack.

**Input**: List of services (name, port, build path), shared infra needed (Postgres/Redis/Kafka/Jaeger)
**Output**: `docker-compose.yml`, `.env.example`, `Makefile` dev targets

## Trigger phrases
- "set up docker-compose"
- "local dev environment"
- "multi-service local setup"
- "add postgres and redis to compose"
- "docker-compose for microservices"
- "I need a local stack for my Go services"

## Output files
- `docker-compose.yml`
- `.env.example`
- Updated `Makefile` with `dev`, `dev-down`, `dev-logs`, `dev-reset` targets

## Source

---
name: go-dev-compose
description: >
  Generates a production-quality docker-compose.yml for local development of
  Go microservices — with health checks, correct dependency ordering, named
  volumes, environment files, and a Makefile with dev/dev-down/logs targets.
  Use this skill whenever the user says "set up docker-compose", "local dev
  environment", "multi-service local setup", "add postgres and redis to
  compose", "docker-compose for microservices", or "I need a local stack for
  my Go services".
---

# Go Dev docker-compose Generator

Generates a well-structured `docker-compose.yml` with health checks, correct
startup ordering, shared infrastructure (Postgres, Redis, Kafka), and a
`make dev` workflow for spinning up the full local stack.

**Input**: List of services (name, port, build path), shared infra needed (Postgres/Redis/Kafka/Jaeger)
**Output**: `docker-compose.yml`, `.env.example`, `Makefile` dev targets

## Workflow

### STEP 1 — Gather config
Ask the user:
- Services to run (name, gRPC port, HTTP port, build path)
- Infrastructure needed: Postgres / Redis / Kafka / Jaeger / all
- Any special env vars per service

### STEP 2 — Generate `docker-compose.yml`

```yaml
version: '3.9'

services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-appuser}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-secret}
      POSTGRES_DB: ${POSTGRES_DB:-appdb}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"]
      interval: 5s
      timeout: 5s
      retries: 10

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 10

  kafka:
    image: confluentinc/cp-kafka:7.6.0
    ports:
      - "9092:9092"
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@localhost:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      CLUSTER_ID: ${KAFKA_CLUSTER_ID:-MkU3OEVBNTcwNTJENDM2Qk}
    healthcheck:
      test: kafka-topics --bootstrap-server localhost:9092 --list
      interval: 10s
      timeout: 10s
      retries: 10

  jaeger:
    image: jaegertracing/all-in-one:1.57
    ports:
      - "16686:16686"  # UI
      - "4317:4317"    # OTLP gRPC
    environment:
      COLLECTOR_OTLP_ENABLED: "true"

  listing-service:
    build:
      context: ./services/listing
      dockerfile: Dockerfile.dev
    ports:
      - "50051:50051"  # gRPC
      - "8080:8080"    # HTTP/metrics
    env_file:
      - .env
    environment:
      DATABASE_URL: postgres://appuser:secret@postgres:5432/appdb?sslmode=disable
      REDIS_ADDR: redis:6379
      KAFKA_BROKERS: kafka:9092
      OTLP_ENDPOINT: jaeger:4317
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      kafka:
        condition: service_healthy
    volumes:
      - ./services/listing:/app  # hot reload with air

volumes:
  postgres_data:
```

### STEP 3 — Generate `.env.example`

```bash
# Postgres
POSTGRES_USER=appuser
POSTGRES_PASSWORD=secret
POSTGRES_DB=appdb

# Kafka
KAFKA_CLUSTER_ID=MkU3OEVBNTcwNTJENDM2Qk

# Service-level
JWT_SECRET=dev-secret-change-in-prod
LOG_LEVEL=debug
ENV=dev
```

### STEP 4 — Generate `Makefile` dev targets

```makefile
dev:
	docker compose up -d --build

docker compose ps

dev-down:
	docker compose down

dev-logs:
	docker compose logs -f --tail=100

dev-reset:
	docker compose down -v --remove-orphans
	docker compose up -d --build

dev-shell-db:
	docker compose exec postgres psql -U $${POSTGRES_USER:-appuser} -d $${POSTGRES_DB:-appdb}
```

### STEP 5 — Write files and `present_files`

## Quality checklist
- [ ] Every service has a `healthcheck` — no race conditions on startup
- [ ] Services use `depends_on: condition: service_healthy` — not just `depends_on`
- [ ] Named volumes for Postgres data — survives `docker compose down` without `-v`
- [ ] `env_file: .env` + explicit env overrides for compose-internal addresses
- [ ] Kafka uses KRaft mode (no ZooKeeper) — simpler for local dev

## Edge cases
- **User wants hot reload**: add `cosmtrek/air` Dockerfile.dev target with `air` entrypoint
- **No Kafka needed**: omit Kafka + Jaeger can be optional too
- **ARM Mac (M1/M2)**: confirm images have `linux/arm64` support; most official images do
- **Podman instead of Docker**: `docker compose` commands are compatible; note `podman-compose` as alternative
