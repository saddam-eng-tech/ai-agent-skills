name: go-request-logging
description: Generates a structured logging setup for Go gRPC microservices with request-ID extraction from gRPC metadata, context injection, and helpers that ensure every log line within a request carries the same trace ID.

# Go Request Logging Setup

Generates a logging interceptor + context helpers so every log line in a request's lifecycle automatically carries `request_id` and `trace_id` — making distributed logs trivially searchable.

**Input**: Logger library (slog/zap/zerolog, default: slog), metadata key for request ID, module path
**Output**: `internal/interceptor/logging.go`, `pkg/logger/context.go`, updated `main.go`

## Trigger phrases
- "add structured logging"
- "request ID propagation"
- "correlate logs across services"
- "logging interceptor"
- "log with context"
- "can't trace a request through logs"
- "set up slog for gRPC"

## Output files
- `pkg/logger/logger.go`
- `pkg/logger/context.go`
- `internal/interceptor/logging.go`
- Updated `main.go` snippet

## Source
`SKILL-19.md`