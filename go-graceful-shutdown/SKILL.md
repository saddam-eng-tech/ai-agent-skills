name: go-graceful-shutdown
description: Generates a graceful shutdown manager for Go microservices that handles OS signals, drains in-flight gRPC requests, closes DB connections, and stops background workers in the correct dependency order with a configurable timeout.

# Go Graceful Shutdown Manager

Generates `pkg/shutdown/manager.go` — registers components in shutdown order and drains them sequentially on SIGTERM/SIGINT, so Kubernetes rolling updates never drop in-flight requests.

**Input**: List of components to shut down (gRPC server, HTTP server, DB pool, background workers)
**Output**: `pkg/shutdown/manager.go`, updated `main.go` with correct shutdown wiring

## Trigger phrases
- "add graceful shutdown"
- "handle SIGTERM"
- "fix dropped requests on deploy"
- "Kubernetes rolling update drops connections"
- "service not shutting down cleanly"
- "goroutines leaked on shutdown"

## Output files
- `pkg/shutdown/manager.go`
- Updated `main.go` snippet showing full shutdown registration
- K8s `terminationGracePeriodSeconds` guidance

## Source
`SKILL-10.md`