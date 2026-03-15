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

---
name: go-goroutine-auditor
description: >
  Reads Go source files and audits every goroutine launch for missing context
  cancellation, missing panic recovery, missing WaitGroup/errgroup, and
  lifecycle management issues — then suggests refactored versions using errgroup
  and goleak. Use this skill whenever the user says "audit my goroutines",
  "goroutine leak", "find naked goroutines", "goroutines not shutting down",
  "structured concurrency in Go", "review my concurrent code", or pastes Go
  code with multiple go func() calls. Also trigger when the user reports
  unexplained memory growth or hanging shutdowns.
---

# Go Goroutine Auditor

Reads one or more Go files, finds every `go func()` / `go someFunc()` launch,
scores each for risk, and generates refactored code using `errgroup` or
`sync.WaitGroup` with proper cancellation and panic recovery.

**Input**: One or more Go source files
**Output**: Audit report (risk per goroutine) + refactored code suggestions + goleak test setup

## Workflow

### STEP 1 — Read the source files
Use `view` on each provided file. If user pastes code directly, work from that.

### STEP 2 — Find all goroutine launches
Scan for:
- `go func()` — anonymous goroutine
- `go someFunc(...)` — named function in goroutine
- `go method.Call(...)` — method call in goroutine

For each launch, record: line number, surrounding function, variables captured.

### STEP 3 — Score each goroutine for risk

| Risk Factor | Check | Score |
|-------------|-------|-------|
| No context | Does goroutine accept/use a context? | +2 |
| No cancellation | Is there a `select { case <-ctx.Done() }` or `ctx.Err()` check? | +2 |
| No panic recovery | Is there a `recover()` or wrapping with errgroup? | +2 |
| No WaitGroup/errgroup | Caller doesn't wait for this goroutine | +1 |
| Captured mutable state | Captures loop variable or shared pointer | +3 |

**Risk levels**: 0–1 = Low, 2–4 = Medium, 5+ = High

### STEP 4 — Generate audit report

```markdown
## Goroutine Audit Report

### File: internal/worker/processor.go

#### Goroutine @ line 42 — Risk: HIGH (8/10)
**Code**: `go func() { process(item) }()`
**Issues**:
- ❌ No context — cannot be cancelled
- ❌ No panic recovery — panic will crash the entire service
- ❌ Captures `item` from loop variable — classic data race
- ❌ No WaitGroup — caller can't wait for completion

**Refactored**:
```go
g, ctx := errgroup.WithContext(ctx)
for _, item := range items {
    item := item  // capture loop variable correctly
    g.Go(func() error {
        return process(ctx, item)
    })
}
if err := g.Wait(); err != nil {
    return err
}
```
```

### STEP 5 — Identify pattern and suggest best fix per case

**Pattern A: Fan-out work items** → use `errgroup`
**Pattern B: Long-running background worker** → use context cancellation + recover()
**Pattern C: Fire-and-forget with cleanup** → use `sync.WaitGroup` + `defer wg.Done()`
**Pattern D: Server goroutine (accept loop)** → wrap in errgroup, handle errors

For Pattern B (background worker):

```go
func runWorker(ctx context.Context, wg *sync.WaitGroup) {
    defer wg.Done()
    defer func() {
        if r := recover(); r != nil {
            slog.Error("worker panic", "recover", r, "stack", debug.Stack())
        }
    }()
    for {
        select {
        case <-ctx.Done():
            return
        default:
            if err := doWork(ctx); err != nil {
                slog.Error("worker error", "err", err)
            }
        }
    }
}
```

### STEP 6 — Generate goleak test setup

```go
func TestMain(m *testing.M) {
    goleak.VerifyTestMain(m,
        goleak.IgnoreTopFunction("google.golang.org/grpc..."),
    )
}
```

### STEP 7 — Present the full report and refactored code

## Output format spec
Summary table at top:
```
| File | Line | Risk | Issues |
|------|------|------|--------|
| processor.go | 42 | HIGH | no ctx, no recover, loop capture |
```
Then full detail + refactored code per goroutine.
End with goleak setup snippet.

## Quality checklist
- [ ] Every goroutine in the file is accounted for in the report
- [ ] Loop variable capture checked (pre-Go 1.22)
- [ ] Refactored code compiles — check imports match
- [ ] goleak ignore list includes known safe goroutines (gRPC, pprof)
- [ ] HIGH risk goroutines flagged prominently at top of report

## Edge cases
- **Go 1.22+**: loop variable capture is fixed by the language; note this and reduce risk score
- **Goroutine is intentionally fire-and-forget**: mark as accepted risk, add a comment to the code
- **errgroup not appropriate** (goroutines with different lifetimes): use context + WaitGroup, explain why
- **Large file with 50+ goroutines**: summarise by pattern first, detail only HIGH risk ones
