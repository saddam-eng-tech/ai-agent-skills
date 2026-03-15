name: go-migration-gen
description: Generates a golang-migrate compatible SQL migration file from a Go domain struct — with correct PostgreSQL types, NOT NULL constraints, indexes, soft delete column, and matching Down migration.

# Go Migration Generator

Reads a Go domain struct and generates a `golang-migrate` SQL migration with correct types, constraints, indexes, soft delete, and a Down rollback.

**Input**: Go struct definition(s), DB dialect (postgres default), table name (optional)
**Output**: `migrations/NNNNNN_create_<entity>.sql` with Up and Down sections

## Trigger phrases
- "generate migration from struct"
- "create SQL migration"
- "add database migration"
- "golang-migrate file"
- "create table for my struct"

## Output files
- `migrations/NNNNNN_create_<entity>.sql` (Up + Down)
- Indexes section after CREATE TABLE
- Updated `Makefile` with migrate targets

## Source

---
name: go-migration-gen
description: >
  Generates a golang-migrate compatible SQL migration file from a Go domain
  struct — with correct PostgreSQL types, NOT NULL constraints, indexes, soft
  delete column, and matching Down migration.
---

## Workflow

### STEP 1 — Read the struct
Extract per field: Field name → snake_case column, Go type → SQL type, pointer = nullable.

### STEP 2 — Go → PostgreSQL type mapping

| Go Type | PostgreSQL Type |
|---------|----------------|
| `string` | `TEXT` |
| `int32` | `INTEGER` |
| `int64` | `BIGINT` |
| `float64` | `NUMERIC(12,4)` |
| `bool` | `BOOLEAN` |
| `time.Time` | `TIMESTAMPTZ` |
| `uuid.UUID` | `UUID` |
| `map[string]interface{}` | `JSONB` |
| Pointer to any above | Same type, nullable |

### STEP 3 — Generate migration file

```sql
-- +migrate Up
CREATE TABLE IF NOT EXISTS <table_name> (
    id           UUID          PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id      UUID          NOT NULL,
    title        TEXT          NOT NULL,
    description  TEXT,
    price        NUMERIC(12,4) NOT NULL DEFAULT 0,
    status       VARCHAR(50)   NOT NULL DEFAULT 'active',
    created_at   TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    updated_at   TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    deleted_at   TIMESTAMPTZ
);

CREATE INDEX idx_<table>_user_id    ON <table_name>(user_id);
CREATE INDEX idx_<table>_status     ON <table_name>(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_<table>_created_at ON <table_name>(created_at DESC) WHERE deleted_at IS NULL;

-- +migrate Down
DROP INDEX IF EXISTS idx_<table>_created_at;
DROP INDEX IF EXISTS idx_<table>_status;
DROP INDEX IF EXISTS idx_<table>_user_id;
DROP TABLE IF EXISTS <table_name>;
```

### STEP 4 — Generate `updated_at` trigger (optional)

```sql
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN NEW.updated_at = NOW(); RETURN NEW; END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER <table>_updated_at
    BEFORE UPDATE ON <table_name>
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

### STEP 5 — Update Makefile

```makefile
migrate:
    migrate -path ./migrations -database "$(DATABASE_URL)" up
migrate-down:
    migrate -path ./migrations -database "$(DATABASE_URL)" down 1
migrate-status:
    migrate -path ./migrations -database "$(DATABASE_URL)" version
```

### STEP 6 — Write file and `present_files`

## Quality checklist
- [ ] Every nullable field (pointer type) omits `NOT NULL`
- [ ] `deleted_at TIMESTAMPTZ` included — enables soft delete pattern
- [ ] Indexes include partial `WHERE deleted_at IS NULL`
- [ ] Down migration drops indexes BEFORE table
- [ ] Money/price fields use `NUMERIC`, never `FLOAT`

## Edge cases
- **Struct has embedded struct**: flatten embedded fields into the same table
- **UUID vs int primary key**: use `SERIAL`/`BIGSERIAL` for int PKs
- **Multiple structs**: generate one migration file per struct, numbered sequentially
- **Struct has no ID field**: generate one: `id UUID PRIMARY KEY DEFAULT gen_random_uuid()`
