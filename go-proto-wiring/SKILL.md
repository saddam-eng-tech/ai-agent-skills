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

---
name: go-proto-wiring
description: >
  Reads a .proto file and generates the full Go handler, service interface,
  and proto-to-domain mapper scaffold — so every RPC method is wired up with
  no silent gaps. Use this skill whenever the user shares a .proto file and
  asks for Go implementation, says "generate handler from proto", "implement
  gRPC service from proto", "wire up my proto file", "stub out handlers", or
  "generate service interface from protobuf". Also trigger when a gRPC service
  has UnimplementedServer methods that need filling in.
---

## Workflow

### STEP 1 — Read the proto file
Use `view` on the provided `.proto` file path. Extract:
- Service name and all RPC methods
- All message types (request, response, domain entities)
- Field names and types for each message

### STEP 2 — Generate `internal/handler/grpc/<service>_handler.go`

```go
type <Service>Handler struct {
    pb.Unimplemented<Service>ServiceServer
    svc service.<Service>Service
}

func (h *<Service>Handler) Create<Entity>(ctx context.Context, req *pb.Create<Entity>Request) (*pb.<Entity>, error) {
    domainEntity := mapper.ToDomain<Entity>(req)
    result, err := h.svc.Create(ctx, domainEntity)
    if err != nil { return nil, pkgerrors.ToGRPC(err) }
    return mapper.ToProto<Entity>(result), nil
}
```

### STEP 3 — Generate `internal/service/<service>_service.go`

```go
type <Service>Service interface {
    Create(ctx context.Context, e *domain.<Entity>) (*domain.<Entity>, error)
    GetByID(ctx context.Context, id string) (*domain.<Entity>, error)
    List(ctx context.Context, filter ListFilter) ([]*domain.<Entity>, error)
    Update(ctx context.Context, e *domain.<Entity>) (*domain.<Entity>, error)
    Delete(ctx context.Context, id string) error
}
```

### STEP 4 — Generate `internal/mapper/<service>_mapper.go`

```go
func ToProto<Entity>(e *domain.<Entity>) *pb.<Entity> {
    if e == nil { return nil }
    return &pb.<Entity>{
        Id: e.ID, Title: e.Title,
        CreatedAt: timestamppb.New(e.CreatedAt),
    }
}

func ToDomain<Entity>(req *pb.Create<Entity>Request) *domain.<Entity> {
    return &domain.<Entity>{Title: req.Title, Status: "active"}
}
```

### STEP 5 — Generate `internal/domain/<entity>.go`

```go
type <Entity> struct {
    ID, UserID, Title, Status string
    Price     float64
    CreatedAt time.Time
    UpdatedAt time.Time
}

func (e *<Entity>) Validate() error {
    if e.Title == "" { return errors.ErrInvalidArgument("title is required") }
    return nil
}
```

### STEP 6 — Write all files and `present_files`

## Quality checklist
- [ ] Every RPC has a corresponding handler method — no silent gaps
- [ ] Handler ONLY calls service — no business logic in handler
- [ ] Mapper ONLY converts types — no business logic in mapper
- [ ] Domain struct has NO proto imports — clean separation
- [ ] `errors.ToGRPC(err)` used in every handler method error path

## Edge cases
- **Proto with nested messages**: generate separate mapper functions per nested type
- **Streaming RPCs**: generate stub with `TODO: implement streaming` comment
- **Existing domain types**: check if `internal/domain/<entity>.go` exists first; don't overwrite
- **Enum fields in proto**: generate Go const block mapping proto enums to domain string constants
