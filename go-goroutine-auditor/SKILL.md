name: go-goroutine-auditor
description: Reads Go source files and audits every goroutine launch for missing context cancellation, missing panic recovery, missing WaitGroup/errgroup, and lifecycle management issues — then suggests refactored versions using errgroup and goleak.

# Go Goroutine Auditor

Reads one or more Go files, finds every `go func()` / `go someFunc()` launch, scores each for risk, and generates refactored code using `errgroup` or `sync.WaitGroup` with proper cancellation and panic recovery.

**Input**: One or more Go source files
**Output**: Audit report (risk per goroutine) + refactored code suggestions + goleak test setup

## Trigger phrases
- "audit my goroutines"
- "goroutine leak"
- "find naked goroutines"
- "goroutines not shutting down"
- "structured concurrency in Go"
- "review my concurrent code"

## Output
- Risk-scored audit report per goroutine (LOW/MEDIUM/HIGH)
- Refactored code using `errgroup` or `sync.WaitGroup`
- `goleak` test setup for CI

## Source
`SKILL-11.md`