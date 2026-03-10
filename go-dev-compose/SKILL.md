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
`SKILL-14.md`