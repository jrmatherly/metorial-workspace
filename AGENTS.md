# Metorial Workspace

Open-source integration platform for agentic AI built on the Model Context Protocol (MCP).

## Package Managers

- **TypeScript**: Bun (platform, index), Yarn (catalog)
- **Go**: Go modules (mcp-engine)

## Core Commands

All tasks orchestrated via [mise](https://mise.jdx.dev):

```bash
mise run dev          # Start all development servers
mise run build        # Build entire workspace
mise run test         # Run all tests
mise tasks            # List all available tasks
```

## AI Context Tools

This workspace uses a 4-Layer Progressive Context Architecture:

| Layer | Tool | Purpose | Activation |
|-------|------|---------|------------|
| 1 | **AGENTS.md** | Essential context | Always loaded (~67 lines) |
| 2 | **Claude Code Skills** | Domain workflows | Auto-triggered by task type |
| 3 | **Serena MCP** | LSP-level code understanding | On-demand via `read_memory()` |
| 4 | **Drift Detect** | Pattern-level AI context | On-demand via `drift_context` |

### Claude Code Skills

The `metorial-claude-plugin/` provides specialized skills for this workspace:

| Skill | Triggers | Purpose |
|-------|----------|---------|
| `mcp-server-development` | "create MCP server", "build server" | MCP SDK patterns, tool design |
| `platform-api-design` | "add endpoint", "Hono route", "Prisma" | Backend API conventions |
| `go-engine-patterns` | "modify engine", "gRPC", "golangci-lint" | Go engine development |

Skills automatically reference Serena memories and Drift patterns for deeper context.

### Serena Memories

For deep context, use Serena memories:

- `workspace_overview` - Structure and navigation
- `development_workflow` - Build, test, and git patterns
- `mise_integration` - Tooling reference
- `claude_code_skills_integration` - Skills architecture documentation

## Repositories

| Repo | Stack | Purpose |
|------|-------|---------|
| `metorial-platform/` | TypeScript + Go | Core platform: API, frontend, MCP engine |
| `metorial/` | TypeScript | MCP server catalog (33+ servers) |
| `metorial-index/` | TypeScript | Server discovery registry |
| `metorial-docs/` | MDX | Documentation (Mintlify) |

Each subdirectory has its own `AGENTS.md` with specific instructions.

## Git Workflow

- **Conventional commits required** - Enforced by `commit-msg` hook
- Branch from `main`, PR to merge
- Cross-repo order: metorial → metorial-index → metorial-platform
- Git hooks auto-configured on workspace entry (via mise)

### Changelog Generation

```bash
mise run changelog:preview   # Preview unreleased changes
mise run changelog:generate  # Generate CHANGELOG.md
mise run changelog:bump      # Calculate next version
```

## Code Style

- **TypeScript**: Strict mode, ESLint + Prettier
- **Go**: golangci-lint v2 (see `mcp-engine/.golangci.yml`)

## Quick Reference

| Task | Command |
|------|---------|
| Install deps | `mise run install` |
| Platform dev | `mise run platform:dev` |
| Engine dev | `mise run engine:dev` |
| Run tests | `mise run test` |
| Lint Go | `mise run engine:lint` |
| Generate Prisma | `mise run platform:prisma:generate` |
| Preview changelog | `mise run changelog:preview` |
| Install git hooks | `mise run git:hooks:install` |
