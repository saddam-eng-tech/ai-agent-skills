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
`SKILL-13.md`