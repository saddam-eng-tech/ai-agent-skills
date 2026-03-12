name: go-resilience-layer
description: Generates a composable resilience layer for Go microservice client calls — circuit breaker, exponential backoff retry with jitter, and token-bucket rate limiter — with optional OpenTelemetry metrics on state transitions.

# Go Resilience Layer

Produces a `pkg/resilience/` package with a composable `Client` wrapper that applies circuit breaker → rate limiter → retry in the correct order for any gRPC or HTTP outbound call.

**Input**: Downstream service name, failure threshold %, retry attempts, RPS limit, module path
**Output**: `pkg/resilience/client.go`, config struct, OTel hooks, unit tests

## Trigger phrases
- "add circuit breaker"
- "retry logic"
- "rate limiting for outbound calls"
- "resilience patterns"
- "my service cascades failures"
- "wrap my gRPC client with retry"

## Output files
- `pkg/resilience/config.go`
- `pkg/resilience/client.go`
- `pkg/resilience/metrics.go` (OTel hooks)
- `pkg/resilience/client_test.go`

## Source
`SKILL-6.md`