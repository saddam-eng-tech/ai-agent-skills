# ai-agent-skills

> A curated collection of battle-tested AI agent skills for **Claude Code** — supercharge your coding agent with plug-and-play skills for React, Go, MCP, and agentic workflows.

[![Skills](https://img.shields.io/badge/skills-36+-blueviolet?style=flat-square)](./README.md)
[![React](https://img.shields.io/badge/React-TypeScript-61DAFB?style=flat-square&logo=react)](./react-scaffold)
[![Go](https://img.shields.io/badge/Go-microservices-00ADD8?style=flat-square&logo=go)](./go-grpc-service-scaffolder)
[![License](https://img.shields.io/github/license/saddam-eng-tech/ai-agent-skills?style=flat-square)](./LICENSE)

---

## What is this?

Stop writing the same boilerplate. Stop repeating yourself to AI.

`ai-agent-skills` is a growing library of **structured skill files** that you attach to **Claude Code** (or any compatible LLM agent) to unlock domain-specific superpowers — from scaffolding production Go gRPC services to auditing every `useEffect` in your React app.

Each skill is a focused, composable instruction set. Think of them as **power-ups for your AI coding agent**.

---

## Quick Start

```bash
git clone https://github.com/saddam-eng-tech/ai-agent-skills.git
```

1. Browse any skill folder below
2. Load the `SKILL.md` into your Claude Code session
3. Trigger it with a natural language prompt — the agent handles the rest

---

## Skills Index

### React & TypeScript

| Skill | What it does |
|-------|-------------|
| [react-scaffold](./react-scaffold) | Generates a full React TS component folder from just a name + description |
| [react-pr-reviewer](./react-pr-reviewer) | Structured code review of React TypeScript PRs — catches what humans miss |
| [react-test-writer](./react-test-writer) | Auto-generates tests for React components |
| [class-to-hooks](./class-to-hooks) | Converts class components to functional hooks with full lifecycle mapping |
| [useeffect-auditor](./useeffect-auditor) | Audits every `useEffect` for dependency array bugs and anti-patterns |
| [component-splitter](./component-splitter) | Finds natural split boundaries in large (100+ line) components |
| [component-docs](./component-docs) | Generates JSDoc for props, functions, and hooks automatically |
| [js-to-ts-migrator](./js-to-ts-migrator) | Migrates JavaScript files to strict TypeScript |
| [ts-error-explainer](./ts-error-explainer) | Explains cryptic TypeScript errors in plain language with fix suggestions |

### Go & Microservices

| Skill | What it does |
|-------|-------------|
| [go-grpc-service-scaffolder](./go-grpc-service-scaffolder) | Scaffolds a production-ready Go gRPC microservice end-to-end |
| [go-grpc-client](./go-grpc-client) | Generates a typed gRPC client factory with retry + timeout wiring |
| [go-grpc-gateway](./go-grpc-gateway) | Adds HTTP-to-gRPC gateway via grpc-gateway with full annotation |
| [go-proto-wiring](./go-proto-wiring) | Reads a .proto file and generates Go handler, service interface & provider |
| [go-otel-setup](./go-otel-setup) | Generates full OpenTelemetry observability package for gRPC services |
| [go-error-mapper](./go-error-mapper) | Creates a typed domain error package that maps errors to gRPC status codes |
| [go-sqlc-repo](./go-sqlc-repo) | Generates type-safe repository layer from SQL schema using sqlc |
| [go-outbox-pattern](./go-outbox-pattern) | Scaffolds the transactional outbox pattern — SQL migrations included |
| [go-cursor-pagination](./go-cursor-pagination) | Reusable cursor-based pagination package with clean API |
| [go-goroutine-auditor](./go-goroutine-auditor) | Audits goroutine launches for missing context cancellation and leaks |
| [go-test-skeleton](./go-test-skeleton) | Generates `_test.go` with gomock stubs from any service interface |
| [go-wire-setup](./go-wire-setup) | Full Google Wire DI setup — providers, injectors, and wiring |
| [go-dev-compose](./go-dev-compose) | Production-quality `docker-compose.yml` for local Go microservice dev |
| [go-resilience-layer](./go-resilience-layer) | Composable resilience layer — circuit breaker, retry, timeout for gRPC clients |
| [go-graceful-shutdown](./go-graceful-shutdown) | Graceful shutdown manager handling OS signals, drain logic and cleanup |
| [go-lint-config](./go-lint-config) | Production-grade `.golangci.yml` config for Go microservices |
| [go-config-validator](./go-config-validator) | Typed config struct with validation and MustLoad() — reads env vars and panics with a clear message on missing/invalid config |
| [go-integration-tests](./go-integration-tests) | Scaffolds a full integration test suite using testcontainers-go with real PostgreSQL, auto-migration, and per-test DB isolation |
| [go-migration-gen](./go-migration-gen) | Generates golang-migrate SQL migration from a Go struct — correct types, constraints, indexes, soft delete, and Down rollback |
| [go-request-logging](./go-request-logging) | Structured logging interceptor for gRPC — injects request_id and trace_id into every log line across a request's lifecycle |

### Agent & Claude Code Workflow

| Skill | What it does |
|-------|-------------|
| [claude-md-generator](./claude-md-generator) | Generates a production `CLAUDE.md` by analyzing your actual codebase |
| [claude-md-auditor](./claude-md-auditor) | Audits existing `CLAUDE.md` against codebase — surfaces stale/wrong instructions |
| [mcp-config-debugger](./mcp-config-debugger) | Diagnoses and fixes MCP configuration errors — broken tools, bad env vars |
| [mcp-tool-pruner](./mcp-tool-pruner) | Detects tool overload in MCP configs and prunes redundant/conflicting tools |
| [context-reset-planner](./context-reset-planner) | Detects context rot in long Claude sessions, produces a clean reset plan |
| [agent-env-syncer](./agent-env-syncer) | Syncs skills + MCP settings into a single source-of-truth workspace config |
| [subagent-scaffolder](./subagent-scaffolder) | Generates complete Claude Code sub-agent definitions — prompts, tools and routing |

### Spec & Documentation

| Skill | What it does |
|-------|-------------|
| [spec-md-generator](./spec-md-generator) | Generates a full `spec.md` through a structured interview with the agent |
| [reverse-spec-writer](./reverse-spec-writer) | Reverse-engineers `spec.md` from an existing codebase |
| [agent-trace-analyzer](./agent-trace-analyzer) | Examines Claude Code agent session logs and traces to explain decisions made, id | 2026-03-12 |
| [rag-gap-auditor](./rag-gap-auditor) | Identifies knowledge gaps in RAG (Retrieval-Augmented Generation) setups by runn | 2026-03-12 |
| [llm-eval-scaffolder](./llm-eval-scaffolder) | Builds LLM evaluation pipelines with structured test cases, LLM-as-judge prompts | 2026-03-12 |

---

## Skill Structure

Each skill follows a consistent folder convention:

```
skill-name/
├── SKILL.md        # The agent instruction file — load this into Claude Code
└── README.md       # Human-readable description + usage (optional)
```

---

## Contributing

Got a skill that saves you time every week? **Share it.**

1. Fork this repo
2. Create your skill folder following the structure above
3. Open a PR — describe what the skill does and how to trigger it

---

## License

[Apache 2.0](./LICENSE) — free to use, fork, and extend.

---

> Built by [@saddam-eng-tech](https://github.com/saddam-eng-tech)
> *"Your agent is only as smart as the skills you give it."*
