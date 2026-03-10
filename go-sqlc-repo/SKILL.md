name: go-sqlc-repo
description: Generates a type-safe repository layer from a SQL schema using sqlc — including sqlc.yaml config, SQL query files, repository interface, pgx/sqlx implementation, and a testify mock.

# Go sqlc Repository Generator

Produces a complete, testable repository layer from a SQL schema — sqlc config, query files, repository interface, and implementation — so you get compile-time safety without an ORM.

**Input**: SQL CREATE TABLE statement(s), Go module path, entity name, DB driver (pgx v5 default)
**Output**: `sqlc.yaml`, `queries/<entity>.sql`, repository interface, pgx implementation, mock

## Trigger phrases
- "set up sqlc"
- "generate repository from schema"
- "add database layer"
- "sqlc wiring"
- "type-safe SQL in Go"
- "replace my ORM with sqlc"

## Output files
- `sqlc.yaml`
- `queries/<entity>.sql`
- `internal/repository/<entity>.go` (interface)
- `internal/repository/postgres/<entity>_repo.go` (implementation)
- `internal/repository/mocks/<entity>_mock.go`
- Updated `Makefile` with `sqlc-gen` target

## Source
`SKILL-4.md`