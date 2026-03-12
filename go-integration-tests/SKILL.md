name: go-integration-tests
description: Scaffolds a full integration test suite for a Go service using testcontainers-go with a real PostgreSQL container, auto-migration runner, per-test database isolation, and example test cases.

# Go Integration Test Setup

Generates a `testhelpers/` package that spins up a real Postgres container per test suite, runs migrations automatically, and provides per-test DB isolation — so your tests catch real SQL bugs that sqlmock misses.

**Input**: Module path, migration directory path, entity/repository names to test
**Output**: `testhelpers/postgres.go`, `testhelpers/migrate.go`, example `*_integration_test.go`

## Trigger phrases
- "set up integration tests"
- "test with real database"
- "testcontainers setup"
- "replace sqlmock with real DB"
- "database integration tests in Go"
- "my unit tests pass but things break in prod"

## Output files
- `testhelpers/postgres.go`
- `testhelpers/migrate.go`
- `testhelpers/isolation.go`
- `internal/repository/postgres/<entity>_repo_integration_test.go`
- Updated `Makefile` with `test-integration` target

## Source
`SKILL-5.md`