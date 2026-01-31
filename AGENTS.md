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

This workspace integrates multiple AI-assistance tools for progressive context:

| Tool | Purpose | Usage |
|------|---------|-------|
| **Serena MCP** | LSP-level code understanding | `read_memory('development_workflow')` |
| **Drift Detect** | Pattern-level AI context | `mise run drift:context` |
| **Mise** | Task orchestration | `mise tasks` |

For deep context, use Serena memories:
- `workspace_overview` - Structure and navigation
- `development_workflow` - Build, test, and git patterns
- `mise_integration` - Tooling reference

## Repositories

| Repo | Stack | Purpose |
|------|-------|---------|
| `metorial-platform/` | TypeScript + Go | Core platform: API, frontend, MCP engine |
| `metorial/` | TypeScript | MCP server catalog (33+ servers) |
| `metorial-index/` | TypeScript | Server discovery registry |
| `metorial-docs/` | MDX | Documentation (Mintlify) |

Each subdirectory has its own `AGENTS.md` with specific instructions.

## Git Workflow

- Conventional commits required
- Branch from `main`, PR to merge
- Cross-repo order: metorial → metorial-index → metorial-platform

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
