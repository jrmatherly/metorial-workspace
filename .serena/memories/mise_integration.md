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
| gita | latest | Multi-repo git operations (context: metorial-workspace) |
| git-cliff | latest | Changelog generation |
| driftdetect | latest | Codebase intelligence |
| golangci-lint | latest | Go linting |
| hk | latest | Git hook manager (mise-hk) |
| pkl | latest | Configuration language for hk |

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
mise run gita:setup             # First-time: register repos + create group
mise run gita:context           # Show/set gita context
mise run git:status             # Status of all repos
mise run git:sync               # Fetch + show status (safe sync)
mise run git:fetch              # Fetch all repos
mise run git:pull               # Pull all repos
mise run git:push               # Push all repos
mise run git:log                # Recent commits across repos
mise run git:diff               # Show uncommitted changes

# Changelog Generation
mise run changelog:preview      # Preview unreleased changes
mise run changelog:generate     # Generate workspace changelog
mise run changelog:bump         # Calculate next version
```

## Task Namespacing

Tasks are namespaced by project:
- `platform:*` - metorial-platform tasks
- `catalog:*` - metorial (catalog) tasks
- `engine:*` - mcp-engine (Go) tasks
- `docs:*` - metorial-docs tasks
- `index:*` - metorial-index tasks
- `git:*` - Multi-repo git operations
- `drift:*` - Codebase intelligence (file-based tasks in `.mise/tasks/drift/`)
- `changelog:*` - Changelog generation and release notes

## Hooks

```toml
[hooks]
# Auto-install tools and hk git hooks on directory entry
enter = "mise install -q && hk install --mise 2>/dev/null || true"
```

## Git Hooks (mise-hk)

Git hooks are managed by [mise-hk](https://github.com/jdx/hk) via `hk.pkl`:

| Hook | Purpose |
|------|---------|
| `pre-commit` | Drift staged file checks |
| `commit-msg` | Conventional commits validation |
| `pre-push` | Drift quality gate |

### Commands

```bash
hk install --mise    # Install hooks (uses mise for tool paths)
hk check             # Check for issues manually
hk fix               # Auto-fix issues
hk validate          # Validate hk.pkl configuration
```

### Claude Code Compatibility

Hooks run silently (output redirected to `/dev/null`) to prevent Claude Code crashes from hk's streaming output. Run `hk check` manually to see issues before committing.

### Tasks

```bash
mise run git:check           # Run hk checks on modified files
mise run git:fix             # Run hk fix on modified files
mise run git:hooks:install   # Install hooks via hk
mise run git:hooks:uninstall # Remove hooks
```

## CI/CD Integration

### Bootstrap Script
A localized bootstrap script (`bin/mise`) is committed to the repo for faster CI:
- Eliminates mise download on every CI run
- Sandboxes mise data into `.mise/` directory
- Version-locked to mise 2026.1.11

```bash
# CI usage (faster than curl)
./bin/mise install
./bin/mise x -- npm test
```

### GitHub Actions
All workflows use `jdx/mise-action@v3` with unified configuration:
- Workspace CI: `.github/workflows/ci.yml`
- Platform workflows: `metorial-platform/.github/workflows/*.yml`

```yaml
- uses: jdx/mise-action@v3
  with:
    version: ${{ env.MISE_VERSION }}
    cache: true
    cache_key_prefix: "mise-v1"
    experimental: true
```

### Drift Quality Gate
CI includes pattern compliance checking via `mise run drift:gate`.

---

## File-Based Tasks

Drift tasks are defined as executable shell scripts in `.mise/tasks/drift/`:

```
.mise/tasks/drift/
├── approve           # Approve patterns interactively
├── approve-all       # Auto-approve ≥95% confidence
├── boundaries        # Data boundaries overview
├── boundaries-check  # Check boundary violations
├── callgraph         # Build call graphs
├── check             # Check staged changes
├── context           # Generate AI context
├── coupling          # Module coupling analysis
├── dashboard         # Open web dashboard
├── error-handling    # Error handling analysis
├── gate              # Quality gate (CI mode)
├── next-steps        # Personalized recommendations
├── scan              # Incremental pattern scan
├── scan-full         # Full pattern scan
├── status            # Pattern status overview
├── test-topology     # Test-to-code mapping
├── trends            # 7-day pattern trends
├── trends-30d        # 30-day pattern trends
├── troubleshoot      # Diagnose issues
└── watch             # Real-time pattern detection
```

Each task runs the corresponding drift command across all projects (MCP Catalog, Platform, Index Registry).

---

**Last Updated**: 2026-02-01
