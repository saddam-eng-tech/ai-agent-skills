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

---
name: go-cursor-pagination
description: >
  Generates a reusable cursor-based pagination package for Go microservices
  with cursor encoding/decoding, SQL builder helper, generic Page[T] response
  type, and gRPC proto additions for page tokens. Use this skill whenever the
  user says "add pagination", "cursor pagination", "page token", "replace
  offset pagination", "pagination is inconsistent", "LIMIT OFFSET is slow",
  "next page token in gRPC", or "infinite scroll backend". Also trigger when
  reviewing List endpoints that use raw LIMIT/OFFSET with no cursor.
---

# Go Cursor Pagination

Generates `pkg/pagination/` — a consistent cursor pagination implementation
that works with any repository, encodes cursors safely, and integrates with
both gRPC (page tokens) and HTTP (query params).

**Input**: Entity name, sort field(s), primary key type (UUID/int64), default page size
**Output**: `pkg/pagination/cursor.go`, `pkg/pagination/page.go`, SQL helper, proto additions

## Workflow

### STEP 1 — Gather config
- Entity name (e.g. `Listing`)
- Sort field (default: `created_at DESC`)
- Cursor field (default: primary key `id`)
- Default page size (default: 20, max: 100)
- Encoding: base64 URL-safe (default) or opaque token

### STEP 2 — Generate `pkg/pagination/cursor.go`

```go
package pagination

import (
    "encoding/base64"
    "encoding/json"
    "fmt"
)

type Cursor struct {
    ID        string    `json:"id"`
    CreatedAt time.Time `json:"created_at"`
}

func (c Cursor) Encode() string {
    b, _ := json.Marshal(c)
    return base64.URLEncoding.EncodeToString(b)
}

func DecodeCursor(s string) (*Cursor, error) {
    if s == "" { return nil, nil }
    b, err := base64.URLEncoding.DecodeString(s)
    if err != nil { return nil, fmt.Errorf("invalid cursor: %w", err) }
    var c Cursor
    if err := json.Unmarshal(b, &c); err != nil {
        return nil, fmt.Errorf("invalid cursor: %w", err)
    }
    return &c, nil
}
```

### STEP 3 — Generate `pkg/pagination/page.go`

```go
type Page[T any] struct {
    Items      []T
    NextCursor string
    HasMore    bool
    Total      *int64
}

func NewPage[T any](items []T, limit int, toCursor func(T) Cursor) Page[T] {
    hasMore := len(items) > limit
    if hasMore { items = items[:limit] }
    p := Page[T]{Items: items, HasMore: hasMore}
    if hasMore {
        p.NextCursor = toCursor(items[len(items)-1]).Encode()
    }
    return p
}
```

### STEP 4 — Generate SQL query helper

```go
func BuildCursorWhere(c *Cursor) (string, map[string]interface{}) {
    if c == nil {
        return "1=1", nil
    }
    return "(created_at, id) < (:cursor_ts, :cursor_id)", map[string]interface{}{
        "cursor_ts": c.CreatedAt,
        "cursor_id": c.ID,
    }
}
```

Example repository method:
```go
func (r *repo) List(ctx context.Context, p pagination.Request) (pagination.Page[*domain.Listing], error) {
    cursor, err := pagination.DecodeCursor(p.PageToken)
    if err != nil { return pagination.Page[*domain.Listing]{}, err }
    where, args := pagination.BuildCursorWhere(cursor)
    query := fmt.Sprintf(`SELECT * FROM listings WHERE deleted_at IS NULL AND %s
        ORDER BY created_at DESC, id DESC LIMIT :limit`, where)
    args["limit"] = p.Limit + 1
    var items []*domain.Listing
    return pagination.NewPage(items, p.Limit, func(l *domain.Listing) pagination.Cursor {
        return pagination.Cursor{ID: l.ID, CreatedAt: l.CreatedAt}
    }), nil
}
```

### STEP 5 — Generate proto additions

```protobuf
message ListListingsRequest {
    int32  page_size  = 1;
    string page_token = 2;
}

message ListListingsResponse {
    repeated Listing listings        = 1;
    string           next_page_token = 2;
}
```

### STEP 6 — Write files and `present_files`

## Output format spec
- `pkg/pagination/cursor.go`
- `pkg/pagination/page.go`
- `pkg/pagination/request.go`
- Updated proto snippet
- Example repository method

## Quality checklist
- [ ] Cursor is opaque to callers
- [ ] `limit+1` trick used — no separate COUNT query needed
- [ ] Tie-breaking on both `created_at` AND `id`
- [ ] Max page size enforced in `Request` validation (default 100)
- [ ] Empty `page_token` = first page — not an error

## Edge cases
- **UUID primary key**: cursor stores UUID string; for int64 PK, encode as int directly
- **Multiple sort fields**: extend Cursor struct and BuildCursorWhere accordingly
- **ASC sort order**: flip `<` to `>` in BuildCursorWhere
- **User wants total count**: add optional `WithTotal` flag; warn about performance cost
