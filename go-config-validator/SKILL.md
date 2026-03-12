name: go-config-validator
description: Generates a typed config struct with validation and MustLoad() for Go microservices — reads environment variables, validates required fields, applies defaults, and panics with a clear message on startup if config is invalid.

# Go Config Validator

Generates `internal/config/config.go` — a typed, validated config struct that reads from environment variables and panics on startup with a clear human-readable error message if required config is missing or invalid.

**Input**: List of env vars (name, type, required/optional, default, description)
**Output**: `internal/config/config.go` with typed struct, validation, MustLoad(), safe Stringer

## Trigger phrases
- "set up config"
- "env var validation"
- "typed config struct"
- "my service crashes with confusing errors on missing env vars"
- "os.Getenv scattered everywhere"
- "add a config layer"

## Output files
- `internal/config/config.go`
- `internal/config/config_test.go`
- `.env.example`
- diff snippet for `main.go`

## Source
`SKILL-7.md`