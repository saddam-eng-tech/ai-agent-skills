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

---
name: go-graceful-shutdown
description: >
  Generates a graceful shutdown manager for Go microservices that handles OS
  signals, drains in-flight gRPC requests, closes DB connections, and stops
  background workers in the correct dependency order with a configurable timeout.
  Use this skill whenever the user says "add graceful shutdown", "handle SIGTERM",
  "fix dropped requests on deploy", "Kubernetes rolling update drops connections",
  "service not shutting down cleanly", or "goroutines leaked on shutdown". Also
  trigger when reviewing main.go files that use os.Exit or have no signal handling.
---

# Go Graceful Shutdown Manager

Generates `pkg/shutdown/manager.go` — registers components in shutdown order
and drains them sequentially on SIGTERM/SIGINT, so Kubernetes rolling updates
never drop in-flight requests.

**Input**: List of components to shut down (gRPC server, HTTP server, DB pool, background workers)
**Output**: `pkg/shutdown/manager.go`, updated `main.go` with correct shutdown wiring

## Workflow

### STEP 1 — Identify components
Ask the user which components need shutdown (or infer from main.go if provided):
- gRPC server (`grpcServer.GracefulStop()`)
- HTTP server (`httpServer.Shutdown(ctx)`)
- DB connection pool (`db.Close()`)
- Background workers (outbox worker, cache warmer, etc.)
- Kafka consumer/producer

Shutdown order MUST be: stop accepting new requests → drain in-flight → close downstream connections.

### STEP 2 — Generate `pkg/shutdown/manager.go`

```go
package shutdown

import (
    "context"
    "os"
    "os/signal"
    "syscall"
    "time"
    "log/slog"
    "sync"
)

type Component struct {
    Name    string
    Timeout time.Duration
    Fn      func(ctx context.Context) error
}

type Manager struct {
    components []Component
    timeout    time.Duration
}

func New(timeout time.Duration) *Manager {
    return &Manager{timeout: timeout}
}

// Register adds a component to the shutdown sequence (LIFO order).
func (m *Manager) Register(name string, timeout time.Duration, fn func(ctx context.Context) error) {
    m.components = append([]Component{{name, timeout, fn}}, m.components...)
}

// Wait blocks until SIGINT or SIGTERM, then shuts down all components.
func (m *Manager) Wait() {
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    sig := <-quit
    slog.Info("shutdown signal received", "signal", sig)

    ctx, cancel := context.WithTimeout(context.Background(), m.timeout)
    defer cancel()

    var wg sync.WaitGroup
    for _, c := range m.components {
        slog.Info("shutting down component", "name", c.Name)
        cCtx, cCancel := context.WithTimeout(ctx, c.Timeout)
        if err := c.Fn(cCtx); err != nil {
            slog.Error("shutdown error", "component", c.Name, "err", err)
        } else {
            slog.Info("component stopped", "name", c.Name)
        }
        cCancel()
        _ = wg
    }
    slog.Info("shutdown complete")
}
```

### STEP 3 — Generate updated `main.go` wiring

Correct shutdown order:
```go
shutdown := pkgshutdown.New(30 * time.Second)

// Register in REVERSE dependency order (first registered = last to shut down)
shutdown.Register("db", 5*time.Second, func(ctx context.Context) error {
    return db.Close()
})
shutdown.Register("outbox-worker", 10*time.Second, func(ctx context.Context) error {
    cancelWorker()
    return nil
})
shutdown.Register("http-server", 5*time.Second, func(ctx context.Context) error {
    return httpServer.Shutdown(ctx)
})
shutdown.Register("grpc-server", 15*time.Second, func(ctx context.Context) error {
    grpcServer.GracefulStop()
    return nil
})

shutdown.Wait()  // blocks until signal
```

### STEP 4 — Generate Kubernetes-aware pre-stop hook note

```go
// Kubernetes sends SIGTERM but may still route traffic for up to 30s.
// Add a sleep BEFORE stopping the gRPC server to drain traffic:
shutdown.Register("pre-stop-sleep", 15*time.Second, func(ctx context.Context) error {
    time.Sleep(10 * time.Second)
    return nil
})
```

### STEP 5 — Write files and `present_files`

## Output format spec
- `pkg/shutdown/manager.go`
- Updated `main.go` snippet showing full shutdown registration
- Comment block explaining K8s terminationGracePeriodSeconds relationship

## Quality checklist
- [ ] gRPC server uses `GracefulStop()` not `Stop()` — waits for in-flight RPCs
- [ ] HTTP server uses `Shutdown(ctx)` not `Close()` — waits for active connections
- [ ] DB closed LAST — services need DB until all requests are drained
- [ ] Background workers cancelled via context, not `os.Exit`
- [ ] Total timeout ≤ `terminationGracePeriodSeconds` in K8s deployment spec

## Edge cases
- **No K8s**: still use graceful shutdown — prevents data corruption on `Ctrl+C` in dev
- **Worker that can't be interrupted**: add a `ForceStop` fallback after timeout
- **Multiple HTTP servers** (health + metrics on different ports): register each separately
- **Kafka producer**: flush pending messages before close — add `producer.Flush(timeout)` call
