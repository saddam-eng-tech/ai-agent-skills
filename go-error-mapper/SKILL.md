name: go-error-mapper
description: Generates a typed domain error package for Go microservices that maps domain errors to correct gRPC status codes and HTTP status codes, with optional stack trace capture.

# Go Domain Error Mapper

Generates a `pkg/errors/` package that defines typed domain errors and maps them to the correct gRPC status codes and HTTP status codes — eliminating the pattern of accidentally returning `codes.Internal` for everything.

**Input**: List of domain error types needed (or use defaults), target transports (gRPC/HTTP/both), module path
**Output**: `pkg/errors/errors.go`, `pkg/errors/grpc.go`, `pkg/errors/http.go`

## Trigger phrases
- "add error handling"
- "map errors to gRPC codes"
- "domain error types"
- "I keep returning Internal for everything"
- "standardise errors across services"
- "how do I return NotFound from gRPC"

## Output files
- `pkg/errors/errors.go` — DomainError type + constructors
- `pkg/errors/grpc.go` — ToGRPC mapping
- `pkg/errors/http.go` — ToHTTP mapping (optional)
- `pkg/errors/errors_test.go` — table-driven tests for every mapping

## Source
`SKILL-3.md`