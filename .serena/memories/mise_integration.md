# Mise-en-Place Integration

## Load When
- Running build/test commands
- Setting up development environment
- Understanding task orchestration

---

## Overview

This workspace uses [mise](https://mise.jdx.dev) for unified tool version management, task orchestration, and auto-dependency management.

## Tool Versions

| Tool | Version | Purpose |
|------|---------|---------|
| Node.js | 24 LTS | Platform frontend/services |
| Bun | 1.3.x | Runtime and package management |
| Go | 1.24 | MCP engine |
| Yarn | 1.x | Catalog package manager |
| gita | latest | Multi-repo git operations |

## Auto-Dependency Management

Mise automatically installs dependencies when lockfiles change:

```toml
# From mise.toml - runs before mise run/x commands
[prepare.bun-platform]
sources = ["metorial-platform/bun.lock"]
outputs = ["metorial-platform/node_modules/"]
run = "bun install"

[prepare.bun-catalog]
sources = ["metorial/bun.lock"]
outputs = ["metorial/node_modules/"]
run = "bun install"

[prepare.bun-index]
sources = ["metorial-index/bun.lock"]
outputs = ["metorial-index/node_modules/"]
run = "bun install"
```

## Common Tasks

```bash
# Development
mise run dev                    # Start all dev servers
mise run platform:dev           # Platform only
mise run engine:dev             # MCP engine with hot reload

# Building
mise run build                  # Build entire workspace
mise run platform:build         # Platform packages
mise run catalog:build          # MCP server catalog
mise run engine:build           # Go binaries

# Testing
mise run test                   # All tests
mise run engine:test            # Go tests
mise run engine:lint            # Go linting

# Multi-repo Git (via gita)
mise run git:status             # Status of all repos
mise run git:fetch              # Fetch all repos
mise run git:pull               # Pull all repos
```

## Task Namespacing

Tasks are namespaced by project:
- `platform:*` - metorial-platform tasks
- `catalog:*` - metorial (catalog) tasks
- `engine:*` - mcp-engine (Go) tasks
- `docs:*` - metorial-docs tasks
- `index:*` - metorial-index tasks
- `git:*` - Multi-repo git operations

## Hooks

```toml
[hooks]
enter = "mise install -q"  # Auto-install tools on directory entry
```

---

**Last Updated**: 2026-01-30
