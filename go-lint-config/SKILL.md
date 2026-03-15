name: go-lint-config
description: Generates a production-grade .golangci.yml configuration for Go microservices with a curated linter set covering security, nil-safety, error handling, and performance — with inline comments explaining every enabled linter.

# Go Lint Config Generator

Generates `.golangci.yml` tuned for production Go microservices — a balanced set of linters covering the issues that actually matter, with every linter explained in an inline comment.

**Input**: Service type (gRPC/HTTP/CLI/library), Go version, strictness (standard/strict)
**Output**: `.golangci.yml` with curated linter set + Makefile lint target + CI config snippet

## Trigger phrases
- "set up golangci-lint"
- "add linting"
- "lint config for Go"
- "golangci-lint config"
- "which linters should I use"
- "configure linting for my Go service"

## Output files
- `.golangci.yml` with every linter annotated
- Updated `Makefile` with lint targets
- GitHub Actions CI snippet

## Source

---
name: go-lint-config
description: >
  Generates a production-grade .golangci.yml configuration for Go microservices
  with a curated linter set covering security, nil-safety, error handling, and
  performance — with inline comments explaining every enabled linter. Use this
  skill whenever the user says "set up golangci-lint", "add linting", "lint config
  for Go", "golangci-lint config", "which linters should I use", or "configure
  linting for my Go service".
---

# Go Lint Config Generator

Generates `.golangci.yml` tuned for production Go microservices — a balanced
set of linters covering the issues that actually matter, with every linter
explained in an inline comment.

**Input**: Service type (gRPC/HTTP/CLI/library), Go version, strictness (standard/strict)
**Output**: `.golangci.yml` with curated linter set + Makefile lint target + CI config snippet

## Workflow

### STEP 1 — Ask for config
- Go version (minimum go1.21)
- Service type: gRPC / HTTP / CLI / library
- Strictness: standard (recommended) / strict (no warnings tolerated)

### STEP 2 — Generate `.golangci.yml`

```yaml
run:
  timeout: 5m
  go: '1.22'

linters:
  disable-all: true
  enable:
    # --- Correctness ---
    - errcheck        # find unchecked errors
    - gosimple        # simplify code
    - govet           # go vet checks (shadow, printf format, etc.)
    - ineffassign     # detect useless assignments
    - staticcheck     # 150+ SA checks (nil deref, unreachable, deprecated)
    - unused          # find unused code

    # --- Security ---
    - gosec           # G101-G601: hard-coded creds, SQL injection, weak crypto

    # --- Error handling ---
    - wrapcheck       # errors from external pkgs must be wrapped
    - errorlint       # correct error wrapping (use errors.Is/As)

    # --- Nil safety ---
    - nilnil          # ban 'return nil, nil' for (T, error) signatures
    - nilerr          # ban returning nil when err != nil

    # --- Style / maintainability ---
    - gofmt           # standard formatting
    - goimports       # import ordering
    - misspell        # catch common typos in comments/strings
    - unconvert       # remove unnecessary type conversions
    - unparam         # find unused function parameters
    - dupl            # flag duplicated code blocks
    - gocritic        # 100+ opinionated but useful checks

    # --- Performance ---
    - prealloc        # suggest slice preallocation
    - noctx           # ban http.Get without context

    # --- Context discipline ---
    - contextcheck    # ensure context is passed through all call chains

linters-settings:
  wrapcheck:
    ignorePackageGlobs:
      - "github.com/myorg/*/pkg/errors"  # our domain errors don't need wrapping
  gosec:
    excludes:
      - G115  # integer overflow in conversion (too noisy)
  gocritic:
    enabled-tags:
      - diagnostic
      - performance
    disabled-checks:
      - hugeParam       # too noisy for domain structs
  dupl:
    threshold: 150

issues:
  exclude-rules:
    - path: '_test\.go'
      linters:
        - errcheck
        - wrapcheck
        - dupl
    - path: 'wire_gen\.go'
      linters: [all]
    - path: 'mock.*\.go'
      linters: [all]
  max-issues-per-linter: 50
  max-same-issues: 10
```

### STEP 3 — Generate `Makefile` lint targets

```makefile
lint:
	golangci-lint run ./...

lint-fix:
	golangci-lint run --fix ./...

lint-new:
	golangci-lint run --new-from-rev=HEAD~1 ./...
```

### STEP 4 — Generate GitHub Actions CI snippet

```yaml
- name: Lint
  uses: golangci/golangci-lint-action@v6
  with:
    version: v1.57.2
    args: --timeout=5m
```

### STEP 5 — Write files and `present_files`

## Quality checklist
- [ ] `disable-all: true` used — opt-in only, no surprise linters
- [ ] Test files excluded from wrapcheck + errcheck — reduces noise
- [ ] Generated files (wire_gen.go, mocks) fully excluded
- [ ] `wrapcheck` ignore list includes the project's own error package
- [ ] Timeout set to 5m — prevents CI hang on large codebases

## Edge cases
- **Library (not service)**: remove `noctx`, `contextcheck`; they're not meaningful for libraries
- **Strict mode**: add `godot`, `godox` (no TODO comments), `exhaustruct`, `cyclop`
- **Legacy codebase with many issues**: add `--new-from-rev` to only lint changed code
- **User wants revive instead of gocritic**: swap them; they overlap significantly
