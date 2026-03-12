name: go-wire-setup
description: Generates a Google Wire dependency injection setup for a Go microservice — provider functions, provider sets, injector function, and wire_gen.go scaffold.

# Go Wire DI Setup

Generates `cmd/server/wire.go` with provider sets and an injector function so Google Wire can auto-generate `wire_gen.go` — replacing tangled manual constructor chains in main.go.

**Input**: List of components and their constructors (or read from main.go), module path
**Output**: `cmd/server/wire.go`, `cmd/server/wire_gen.go` scaffold, Makefile target

## Trigger phrases
- "set up Wire"
- "dependency injection in Go"
- "wire providers"
- "google wire setup"
- "generate wire.go"
- "DI for my Go service"
- "my main.go is getting too large to manage dependencies"

## Output files
- `cmd/server/wire.go` (provider sets + injector)
- `cmd/server/app.go` (App struct + NewApp)
- Updated `main.go` snippet
- Updated `Makefile` with `wire-gen` target

## Source
`SKILL-18.md`