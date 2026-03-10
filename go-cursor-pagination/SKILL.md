name: go-cursor-pagination
description: Generates a reusable cursor-based pagination package for Go microservices with cursor encoding/decoding, SQL builder helper, generic Page[T] response type, and gRPC proto additions for page tokens.

# Go Cursor Pagination

Generates `pkg/pagination/` — a consistent cursor pagination implementation that works with any repository, encodes cursors safely, and integrates with both gRPC (page tokens) and HTTP (query params).

**Input**: Entity name, sort field(s), primary key type (UUID/int64), default page size
**Output**: `pkg/pagination/cursor.go`, `pkg/pagination/page.go`, SQL helper, proto additions

## Trigger phrases
- "add pagination"
- "cursor pagination"
- "page token"
- "replace offset pagination"
- "LIMIT OFFSET is slow"
- "next page token in gRPC"
- "infinite scroll backend"

## Output files
- `pkg/pagination/cursor.go`
- `pkg/pagination/page.go`
- `pkg/pagination/request.go`
- Updated proto snippet
- Example repository method

## Source
`SKILL-12.md`