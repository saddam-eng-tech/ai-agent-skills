name: go-test-skeleton
description: Reads a Go service interface and generates a complete _test.go file with gomock setup and 3 test cases per method — success, validation error, and downstream error.

# Go Test Skeleton Generator

Reads a Go service interface and produces a complete `_test.go` covering every method with success, validation-failure, and downstream-error cases — using gomock for mocking dependencies.

**Input**: A Go file containing a service interface (or paste the interface directly)
**Output**: `<service>_test.go` with gomock setup, test cases per method, test helpers

## Trigger phrases
- "generate unit tests"
- "write tests for this interface"
- "create service tests"
- "add test coverage"
- "gomock test setup"
- "scaffold tests from interface"

## Output files
- `internal/service/<entity>_service_test.go`
- `//go:generate` directive for mockgen
- Minimum 3 test functions per interface method (success, validation failure, repo error)

## Source
`SKILL-15.md`