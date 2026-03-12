# ai-agent-skills
AI agent skills for Claude code AI


## Skills

| Skill | Description | Added |
|-------|-------------|-------|
| [react-pr-reviewer](./react-pr-reviewer) | Performs a structured code review of React TypeScript pull requests or | 2026-03-08 |
| [js-to-ts-migrator](./js-to-ts-migrator) | > | 2026-03-08 |
| [class-to-hooks](./class-to-hooks) | Converts a React class component to a functional component using hooks, mapping  | 2026-03-09 |
| [useeffect-auditor](./useeffect-auditor) | Analyses React component files and audits every useEffect hook for dependency ar | 2026-03-09 |
| [react-scaffold](./react-scaffold) | Generates a complete React TypeScript component folder from a name and descripti | 2026-03-09 |
| [component-splitter](./component-splitter) | Analyses a large (100+ line) React component, identifies natural split boundarie | 2026-03-09 |
| [component-docs](./component-docs) | Generates JSDoc comments for a React TypeScript component's props, functions, an | 2026-03-09 |
| [go-grpc-service-scaffolder](./go-grpc-service-scaffolder) | Scaffolds a complete, production-ready Go gRPC microservice following Carousell- | 2026-03-09 |
| [go-otel-setup](./go-otel-setup) | Generates a complete OpenTelemetry observability package for a Go gRPC microserv | 2026-03-10 |
| [go-error-mapper](./go-error-mapper) | Generates a typed domain error package for Go microservices that maps domain err | 2026-03-10 |
| [go-sqlc-repo](./go-sqlc-repo) | Generates a type-safe repository layer from a SQL schema using sqlc — includin | 2026-03-10 |
| [go-outbox-pattern](./go-outbox-pattern) | Scaffolds the transactional outbox pattern for a Go microservice — SQL migrati | 2026-03-10 |
| [go-grpc-client](./go-grpc-client) | Generates a typed gRPC client factory for calling internal Go microservices with | 2026-03-10 |
| [go-goroutine-auditor](./go-goroutine-auditor) | Reads Go source files and audits every goroutine launch for missing context canc | 2026-03-10 |
| [go-cursor-pagination](./go-cursor-pagination) | Generates a reusable cursor-based pagination package for Go microservices with c | 2026-03-10 |
| [go-dev-compose](./go-dev-compose) | Generates a production-quality docker-compose.yml for local development of Go mi | 2026-03-10 |
| [go-test-skeleton](./go-test-skeleton) | Reads a Go service interface and generates a complete _test.go file with gomock  | 2026-03-10 |
| [go-proto-wiring](./go-proto-wiring) | Reads a .proto file and generates the full Go handler, service interface, and pr | 2026-03-10 |
| [go-wire-setup](./go-wire-setup) | Generates a Google Wire dependency injection setup for a Go microservice — pro | 2026-03-10 |
| [go-grpc-gateway](./go-grpc-gateway) | Generates an HTTP-to-gRPC gateway using grpc-gateway — adds HTTP binding annot | 2026-03-10 |
| [claude-md-generator](./claude-md-generator) | Generates a production-ready CLAUDE.md file for any project by analyzing the cod | 2026-03-12 |
| [claude-md-auditor](./claude-md-auditor) | Audits an existing CLAUDE.md file against the actual codebase to find stale comm | 2026-03-12 |
| [mcp-config-debugger](./mcp-config-debugger) | Diagnoses and fixes MCP (Model Context Protocol) configuration errors including  | 2026-03-12 |
| [mcp-tool-pruner](./mcp-tool-pruner) | Audits an MCP server configuration to identify tool overload — too many tools  | 2026-03-12 |
| [context-reset-planner](./context-reset-planner) | Detects context rot in long LLM/Claude Code sessions and produces a structured c | 2026-03-12 |
| [agent-env-syncer](./agent-env-syncer) | Generates a single source-of-truth workspace config that syncs skills and MCP se | 2026-03-12 |
| [spec-md-generator](./spec-md-generator) | Generates a complete spec.md (project specification document) by having a struct | 2026-03-12 |
