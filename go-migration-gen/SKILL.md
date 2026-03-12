name: go-migration-gen
description: Generates a golang-migrate compatible SQL migration file from a Go domain struct — with correct PostgreSQL types, NOT NULL constraints, indexes, soft delete column, and matching Down migration.

# Go Migration Generator

Reads a Go domain struct and generates a `golang-migrate` SQL migration with correct types, constraints, indexes, soft delete, and a Down rollback — so the schema matches the struct without manual translation.

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
`SKILL-17.md`