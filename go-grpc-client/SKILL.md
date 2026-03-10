name: go-grpc-client
description: Generates a typed gRPC client factory for calling internal Go microservices with connection pooling, TLS toggle, client-side interceptors (logging, OTel tracing, auth header injection), and a health-check poller.

# Go gRPC Client Factory

Generates `pkg/clients/<service>_client.go` — a typed gRPC client wrapper that reuses connections, propagates auth and trace context, and logs every outbound call.

**Input**: Target service name, proto package path, auth token env var, TLS cert path or insecure flag, module path
**Output**: `pkg/clients/<service>_client.go` with factory, config, interceptors, and typed wrapper methods

## Trigger phrases
- "create a gRPC client"
- "call another microservice"
- "set up service-to-service gRPC"
- "add client interceptors"
- "gRPC connection setup"
- "how do I call the listing service from the order service"

## Output files
- `pkg/clients/<service>_client.go`
- `pkg/clients/<service>_interceptors.go`
- `pkg/clients/<service>_health.go` (optional)
- Updated `main.go` snippet (client init + close)

## Source
`SKILL-9.md`