name: go-grpc-service-scaffolder
description: Scaffolds a complete, production-ready Go gRPC microservice following Carousell-style Clean Architecture (handler → service → repository layers), with Protobuf definition, Dockerfile, health check, gRPC interceptors (logging, recovery, tracing), dependency injection via Wire, and structured project layout. Use this skill whenever the user asks to "scaffold a Go service", "create a new Go microservice", "generate gRPC boilerplate", "set up a Go backend service", "create a Go service like Carousell", or says things like "I need a new Go service", "bootstrap a Go gRPC server", "generate Go service skeleton", or "help me start a new microservice in Go". Also trigger when user shares a .proto file and wants a full service implementation, or when they describe a domain (e.g. "listing service", "payment service", "user service") and ask for a Go backend.

# Go gRPC Service Scaffolder

Generates a complete Go gRPC microservice following Clean Architecture
as practised at companies like Carousell — where services communicate via
gRPC internally, expose HTTP through an API Gateway, and use PostgreSQL
with a clear separation of transport / business / data layers.


## Architecture Reference (Carousell-style)

```
clients (web/mobile)
        │
        ▼ HTTP
   [API Gateway]
        │
        ▼ gRPC
  ┌─────────────┐   gRPC   ┌─────────────┐
  │  Service A  │─────────▶│  Service B  │
  │  (e.g. OMS) │          │  (e.g. PMS) │
  └─────┬───────┘          └─────────────┘
        │
   PostgreSQL / Redis
```

Each service owns:
- Its own Protobuf contract (`.proto`)
- Clean layers: `handler` → `service` → `repository`
- Dependency injection (Wire or manual)
- Interceptors for logging, panic recovery, and distributed tracing
- A `/healthz` HTTP endpoint alongside the gRPC server


## Step-by-Step Workflow

### STEP 1 — Gather Requirements

Ask (or infer from context) these key questions:
1. **Service name** — e.g. `listing-service`, `payment-service`
2. **Domain entities** — e.g. `Listing`, `Order`, `User`
3. **RPC methods** — e.g. `CreateListing`, `GetListing`, `SearchListings`
4. **Database** — PostgreSQL (default), MySQL, or none
5. **Auth** — JWT interceptor needed? (default: yes)
6. **Module path** — Go module path, e.g. `github.com/myorg/listing-service`

If the user provides a `.proto` file, parse it and skip to STEP 3.


### STEP 2 — Generate the `.proto` File

Produce a `proto/v1/<service>.proto` matching what the user described.

**Template:**
```protobuf
syntax = "proto3";

package <package>.v1;
option go_package = "<module>/gen/proto/v1;<package>v1";

import "google/protobuf/timestamp.proto";
import "google/protobuf/empty.proto";

service <ServiceName>Service {
  rpc Create<Entity>(Create<Entity>Request)  returns (<Entity>);
  rpc Get<Entity>(Get<Entity>Request)        returns (<Entity>);
  rpc List<Entity>s(List<Entity>sRequest)    returns (List<Entity>sResponse);
  rpc Update<Entity>(Update<Entity>Request)  returns (<Entity>);
  rpc Delete<Entity>(Delete<Entity>Request)  returns (google.protobuf.Empty);
}

message <Entity> {
  string id         = 1;
  string user_id    = 2;
  // ... domain fields ...
  google.protobuf.Timestamp created_at = 10;
  google.protobuf.Timestamp updated_at = 11;
}

// ... request/response messages ...
```


### STEP 3 — Scaffold the Directory Structure

Generate ALL files below. Read `references/file-templates.md` for the
exact code content of each file.

```
<service-name>/
├── cmd/
│   └── server/
│       └── main.go                  # Wire up and start both gRPC + HTTP servers
├── proto/
│   └── v1/
│       └── <service>.proto          # Protobuf contract
├── gen/
│   └── proto/v1/                    # Generated protobuf Go code (gitignored, regenerated)
├── internal/
│   ├── handler/
│   │   └── grpc/
│   │       └── <entity>_handler.go  # gRPC transport layer — calls service
│   ├── service/
│   │   ├── <entity>_service.go      # Business logic interface + implementation
│   │   └── <entity>_service_test.go # Unit tests (mocked repository)
│   ├── repository/
│   │   ├── <entity>_repository.go   # Repository interface
│   │   └── postgres/
│   │       └── <entity>_repo.go     # PostgreSQL implementation
│   ├── domain/
│   │   └── <entity>.go              # Domain structs (NOT proto types)
│   └── interceptor/
│       ├── logging.go               # Structured request/response logging
│       ├── recovery.go              # Panic recovery → gRPC Internal error
│       └── auth.go                  # JWT token validation (if auth=true)
├── pkg/
│   ├── errors/
│   │   └── errors.go                # Domain errors → gRPC status codes
│   └── health/
│       └── handler.go               # HTTP /healthz + /readyz
├── migrations/
│   └── 000001_create_<entity>.sql   # Initial schema migration
├── Dockerfile                       # Multi-stage build
├── docker-compose.yml               # Local dev: service + postgres + redis
├── Makefile                         # proto-gen, build, test, migrate targets
├── go.mod
└── README.md
```


### STEP 4 — Generate Each File

See reference files for complete implementations:
- **postgres-repo.md** - Full PostgreSQL repository implementation with sqlx
- **interceptors.md** - Complete gRPC interceptors (logging, recovery, auth)

#### `cmd/server/main.go`
```go
package main

import (
    "context"
    "net"
    "net/http"
    "os"
    "os/signal"
    "syscall"

    "google.golang.org/grpc"
    "google.golang.org/grpc/reflection"

    "<module>/internal/handler/grpc"
    "<module>/internal/interceptor"
    "<module>/internal/repository/postgres"
    "<module>/internal/service"
    "<module>/pkg/health"
    pb "<module>/gen/proto/v1"
)

func main() {
    ctx := context.Background()

    // --- Dependencies (manual DI — replace with Wire for large services) ---
    db := postgres.MustConnect(os.Getenv("DATABASE_URL"))
    repo := postgres.New<Entity>Repo(db)
    svc  := service.New<Entity>Service(repo)
    h    := handler.New<Entity>Handler(svc)

    // --- gRPC Server ---
    grpcServer := grpc.NewServer(
        grpc.ChainUnaryInterceptor(
            interceptor.Recovery(),
            interceptor.Logging(),
            interceptor.Auth(os.Getenv("JWT_SECRET")),
        ),
    )
    pb.Register<ServiceName>ServiceServer(grpcServer, h)
    reflection.Register(grpcServer)

    lis, _ := net.Listen("tcp", ":"+os.Getenv("GRPC_PORT"))
    go grpcServer.Serve(lis)

    // --- HTTP health server ---
    mux := http.NewServeMux()
    health.Register(mux)
    go http.ListenAndServe(":"+os.Getenv("HTTP_PORT"), mux)

    // --- Graceful shutdown ---
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    grpcServer.GracefulStop()
}
```

#### `internal/handler/grpc/<entity>_handler.go` (Transport Layer)
```go
// Translates gRPC proto ↔ domain types. No business logic here.
type <Entity>Handler struct {
    pb.Unimplemented<ServiceName>ServiceServer
    svc service.<Entity>Service
}

func (h *<Entity>Handler) Create<Entity>(
    ctx context.Context,
    req *pb.Create<Entity>Request,
) (*pb.<Entity>, error) {
    entity, err := h.svc.Create(ctx, toDomain(req))
    if err != nil {
        return nil, toGRPCError(err)
    }
    return toProto(entity), nil
}
```

#### `internal/service/<entity>_service.go` (Business Layer)
```go
// Interface-first — makes mocking trivial in tests
type <Entity>Service interface {
    Create(ctx context.Context, e *domain.<Entity>) (*domain.<Entity>, error)
    GetByID(ctx context.Context, id string) (*domain.<Entity>, error)
    List(ctx context.Context, filter ListFilter) ([]*domain.<Entity>, error)
    Update(ctx context.Context, e *domain.<Entity>) (*domain.<Entity>, error)
    Delete(ctx context.Context, id string) error
}

type <entity>ServiceImpl struct {
    repo repository.<Entity>Repository
}

func (s *<entity>ServiceImpl) Create(ctx context.Context, e *domain.<Entity>) (*domain.<Entity>, error) {
    // validate
    if err := e.Validate(); err != nil {
        return nil, errors.ErrInvalidArgument(err.Error())
    }
    e.ID = uuid.NewString()
    e.CreatedAt = time.Now()
    return s.repo.Create(ctx, e)
}
```

#### `pkg/errors/errors.go` (Domain → gRPC status mapping)
```go
// Carousell pattern: typed domain errors map to gRPC codes
type DomainError struct {
    Code    codes.Code
    Message string
}

var (
    ErrNotFound        = &DomainError{codes.NotFound, "resource not found"}
    ErrAlreadyExists   = &DomainError{codes.AlreadyExists, "resource already exists"}
    ErrInvalidArgument = func(msg string) *DomainError {
        return &DomainError{codes.InvalidArgument, msg}
    }
    ErrUnauthenticated = &DomainError{codes.Unauthenticated, "invalid credentials"}
)

func ToGRPCStatus(err error) error {
    var de *DomainError
    if errors.As(err, &de) {
        return status.Error(de.Code, de.Message)
    }
    return status.Error(codes.Internal, "internal error")
}
```

#### `Dockerfile` (Multi-stage)
```dockerfile
# Build stage
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -ldflags="-s -w" -o /bin/server ./cmd/server

# Runtime stage — distroless for minimal attack surface
FROM gcr.io/distroless/static:nonroot
COPY --from=builder /bin/server /server
EXPOSE 50051 8080
ENTRYPOINT ["/server"]
```

#### `Makefile`
```makefile
PROTO_DIR = proto/v1
GEN_DIR   = gen/proto/v1

.PHONY: proto build test migrate lint

proto:
	protoc --go_out=$(GEN_DIR) --go_opt=paths=source_relative \
	       --go-grpc_out=$(GEN_DIR) --go-grpc_opt=paths=source_relative \
	       $(PROTO_DIR)/*.proto

build:
	go build -o bin/server ./cmd/server

test:
	go test -race -cover ./...

migrate:
	migrate -path migrations -database "$(DATABASE_URL)" up

lint:
	golangci-lint run ./...
```


### STEP 5 — Generate Unit Test Skeleton

Always generate `internal/service/<entity>_service_test.go` using
`gomock` or `testify/mock`:

```go
func TestCreate<Entity>_Success(t *testing.T) {
    ctrl := gomock.NewController(t)
    mockRepo := mocks.NewMock<Entity>Repository(ctrl)

    mockRepo.EXPECT().
        Create(gomock.Any(), gomock.Any()).
        Return(&domain.<Entity>{ID: "abc-123"}, nil)

    svc := service.New<Entity>Service(mockRepo)
    got, err := svc.Create(context.Background(), &domain.<Entity>{...})

    assert.NoError(t, err)
    assert.Equal(t, "abc-123", got.ID)
}

func TestCreate<Entity>_ValidationFailure(t *testing.T) { ... }
func TestCreate<Entity>_RepositoryError(t *testing.T) { ... }
```


### STEP 6 — Output All Files

Write every generated file to disk. Then present:
1. The full directory tree
2. All generated files
3. A "Getting Started" block:

```
## Getting Started

1.  Install dependencies:
    go mod tidy

2.  Generate protobuf code:
    make proto

3.  Start local stack:
    docker-compose up -d

4.  Run migrations:
    make migrate

5.  Start the service:
    GRPC_PORT=50051 HTTP_PORT=8080 DATABASE_URL=... go run ./cmd/server

6.  Test with grpcurl:
    grpcurl -plaintext localhost:50051 list
```


## Carousell Architecture Patterns — Quick Reference

| Pattern | Implementation |
|---------|---------------|
| Internal comms | gRPC with proto contracts |
| External API | HTTP via API Gateway (not service-direct) |
| DB default | PostgreSQL — "cost-effective for most cases" |
| Error handling | Domain errors → `pkg/errors` → gRPC status codes |
| DI | Manual constructor injection (Wire for large services) |
| Concurrency | Goroutines + context cancellation (never raw goroutines without sync) |
| Distributed txn | TCC (Try-Confirm-Cancel) for cross-service state changes |
| Import cycle fix | Keep interfaces + structs in same package; avoid circular deps |
| QPS target | Single Go service handles 5000+ QPS with p99 < 50ms |
| Observability | Request ID propagation via gRPC metadata + structured logging |


## Reference Files

This skill includes reference implementation files:

### postgres-repo.md
Complete PostgreSQL repository implementation using sqlx with:
- CRUD operations (Create, FindByID, List, Update, Delete)
- Soft delete pattern (Carousell standard)
- Connection pooling configuration
- SQL migration template

### interceptors.md  
Production-ready gRPC interceptors:
- **Logging** - Structured request/response logging with request ID
- **Recovery** - Panic recovery with stack traces
- **Auth** - JWT token validation with context injection