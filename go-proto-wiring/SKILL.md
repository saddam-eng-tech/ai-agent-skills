name: go-proto-wiring
description: Reads a .proto file and generates the full Go handler, service interface, and proto-to-domain mapper scaffold — so every RPC method is wired up with no silent gaps.

# Go Proto → Handler Wiring

Reads a `.proto` file and generates the full handler, service interface, and mapper layer — one file per concern, with every RPC method stubbed and ready to implement.

**Input**: A `.proto` file (path or paste), Go module path, domain package path
**Output**: `handler.go`, `service.go` (interface), `mapper.go` (proto↔domain converters)

## Trigger phrases
- "generate handler from proto"
- "implement gRPC service from proto"
- "wire up my proto file"
- "stub out handlers"
- "generate service interface from protobuf"

## Output files
- `internal/handler/grpc/<service>_handler.go`
- `internal/service/<service>_service.go` (interface + stub)
- `internal/mapper/<service>_mapper.go`
- `internal/domain/<entity>.go`

## Source
`SKILL-16.md`