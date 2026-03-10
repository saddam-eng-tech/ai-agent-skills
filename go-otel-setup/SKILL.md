name: go-otel-setup
description: Generates a complete OpenTelemetry observability package for a Go gRPC microservice — traces (Jaeger/OTLP), Prometheus metrics, structured logging with trace-ID injection, and gRPC interceptors that auto-propagate spans.

# Go OpenTelemetry Setup

Wires OpenTelemetry SDK (traces + metrics) and structured logging into a Go gRPC service so every request is automatically instrumented end-to-end.

**Input**: Service name, OTLP endpoint, module path, signals wanted (traces/metrics/logs)
**Output**: `pkg/observability/` package, gRPC interceptors, Prometheus endpoint, docker-compose additions

## Trigger phrases
- "add observability"
- "set up OpenTelemetry"
- "add tracing to my Go service"
- "instrument my gRPC service"
- "add Prometheus metrics"
- "set up OTel"
- "I can't correlate logs across services"

## Output files
- `pkg/observability/provider.go`
- `pkg/observability/metrics.go`
- `pkg/observability/interceptor.go`
- `pkg/observability/logger.go`
- `prometheus.yml`
- diff snippet for `main.go` and `docker-compose.yml`

## Source
`SKILL-2.md`