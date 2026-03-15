name: go-grpc-gateway
description: Generates an HTTP-to-gRPC gateway using grpc-gateway — adds HTTP binding annotations to a .proto file, generates the gateway server with mux, CORS config, auth passthrough, and an OpenAPI/Swagger spec.

# Go gRPC-Gateway Generator

Takes a `.proto` file and generates the full grpc-gateway setup — HTTP annotations, gateway server, CORS, auth passthrough, and Swagger spec.

**Input**: `.proto` file, HTTP path prefix, auth header to forward, CORS allowed origins
**Output**: Annotated `.proto`, `cmd/gateway/main.go`, `api/swagger.json`, updated Makefile

## Trigger phrases
- "add HTTP gateway"
- "grpc-gateway setup"
- "expose gRPC as REST"
- "HTTP to gRPC"
- "add REST API to my gRPC service"
- "generate OpenAPI from proto"
- "clients need HTTP not gRPC"

## Output files
- Updated `.proto` with HTTP annotations
- `cmd/gateway/main.go`
- `cmd/gateway/Dockerfile`
- Updated `Makefile` proto target
- `docker-compose` addition

## Source

---
name: go-grpc-gateway
description: >
  Generates an HTTP-to-gRPC gateway using grpc-gateway.
---

## Workflow

### STEP 1 — Read the proto file
Extract service name, all RPC methods, and message types.

### STEP 2 — Add HTTP annotations to proto

```protobuf
import "google/api/annotations.proto";

service ListingService {
  rpc CreateListing(CreateListingRequest) returns (Listing) {
    option (google.api.http) = { post: "/api/v1/listings" body: "*" };
  }
  rpc GetListing(GetListingRequest) returns (Listing) {
    option (google.api.http) = { get: "/api/v1/listings/{id}" };
  }
  rpc ListListings(ListListingsRequest) returns (ListListingsResponse) {
    option (google.api.http) = { get: "/api/v1/listings" };
  }
  rpc UpdateListing(UpdateListingRequest) returns (Listing) {
    option (google.api.http) = { patch: "/api/v1/listings/{id}" body: "*" };
  }
  rpc DeleteListing(DeleteListingRequest) returns (google.protobuf.Empty) {
    option (google.api.http) = { delete: "/api/v1/listings/{id}" };
  }
}
```

### STEP 3 — Generate `cmd/gateway/main.go`

```go
func main() {
    ctx := context.Background()
    mux := runtime.NewServeMux(
        runtime.WithIncomingHeaderMatcher(authHeaderMatcher),
    )
    opts := []grpc.DialOption{grpc.WithTransportCredentials(insecure.NewCredentials())}
    pb.RegisterListingServiceHandlerFromEndpoint(ctx, mux, os.Getenv("LISTING_SERVICE_ADDR"), opts)
    http.ListenAndServe(":"+os.Getenv("GATEWAY_PORT"), corsMiddleware(mux))
}

func authHeaderMatcher(key string) (string, bool) {
    switch key {
    case "Authorization", "X-Request-Id": return key, true
    }
    return runtime.DefaultHeaderMatcher(key)
}

func corsMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Access-Control-Allow-Origin", os.Getenv("CORS_ALLOWED_ORIGINS"))
        w.Header().Set("Access-Control-Allow-Methods", "GET, POST, PATCH, DELETE, OPTIONS")
        w.Header().Set("Access-Control-Allow-Headers", "Authorization, Content-Type, X-Request-Id")
        if r.Method == http.MethodOptions { w.WriteHeader(http.StatusNoContent); return }
        next.ServeHTTP(w, r)
    })
}
```

### STEP 4 — Update Makefile for proto generation

```makefile
proto:
    protoc -I . -I $(GOPATH)/pkg/mod/github.com/grpc-ecosystem/grpc-gateway/v2@v2.19.0/third_party/googleapis \
        --go_out=gen/proto/v1 --go_opt=paths=source_relative \
        --go-grpc_out=gen/proto/v1 --go-grpc_opt=paths=source_relative \
        --grpc-gateway_out=gen/proto/v1 --grpc-gateway_opt=paths=source_relative \
        --openapiv2_out=api \
        proto/v1/*.proto
```

### STEP 5 — Add gateway to docker-compose

```yaml
  gateway:
    environment:
      GATEWAY_PORT: 8080
      LISTING_SERVICE_ADDR: listing-service:50051
      CORS_ALLOWED_ORIGINS: "http://localhost:3000"
    ports:
      - "8080:8080"
    depends_on:
      listing-service:
        condition: service_healthy
```

### STEP 6 — Write all files and `present_files`

## Quality checklist
- [ ] Every RPC has exactly one HTTP method annotation
- [ ] Path parameters (`{id}`) match field names in request message
- [ ] `Authorization` header forwarded to gRPC metadata
- [ ] CORS origins configurable via env var — not hardcoded
- [ ] `OPTIONS` pre-flight handled for browser clients

## Edge cases
- **Multiple services**: register each service handler in the same mux
- **Request ID propagation**: add `X-Request-Id` to `authHeaderMatcher`
- **Auth at gateway vs service**: recommend gateway for external, service for internal
- **gRPC streaming**: grpc-gateway supports server-streaming via HTTP chunked transfer
