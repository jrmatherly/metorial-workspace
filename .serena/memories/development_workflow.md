# Development Workflow

## Load When
- Starting development tasks
- Understanding build/test patterns
- Contributing to any repository

---

## Getting Started

```bash
# 1. Enter workspace (auto-installs tools via mise hook)
cd /path/to/metorial-workspace

# 2. Dependencies auto-install when needed, or manually:
mise prep

# 3. Start development
mise run dev
```

## Repository-Specific Workflows

### metorial (MCP Server Catalog)

```bash
# Add new MCP server (interactive wizard)
mise run catalog:add-server

# Build all servers
mise run catalog:build

# Check version consistency
mise run catalog:check-versions
```

**Server Structure**:
```
servers/<name>/
├── server.ts         # Implementation
├── metorial.json     # Server metadata
├── package.json      # Dependencies
└── tsconfig.json     # TypeScript config
```

### metorial-platform

```bash
# Development mode
mise run platform:dev

# Build all packages
mise run platform:build

# Run tests
mise run platform:test

# Database operations
mise run platform:prisma:generate
mise run platform:prisma:push
```

### mcp-engine (Go)

```bash
# Development with hot reload
mise run engine:dev

# Build binary
mise run engine:build

# Run tests
mise run engine:test

# Linting
mise run engine:lint

# Generate protobuf
mise run engine:proto
```

### metorial-docs

```bash
# Local preview
mise run docs:dev

# Build documentation
mise run docs:build
```

## Git Workflow

Each repository is independent but coordinated:

```bash
# Check status across all repos
mise run git:status

# Fetch/pull all repos
mise run git:fetch
mise run git:pull

# Checkout branch in all repos
mise run git:checkout <branch>
```

**Commit Order** (when making cross-repo changes):
1. metorial (if SDK changes)
2. metorial-index (if registry changes)
3. metorial-platform (if platform changes)
4. metorial-docs (documentation last)

## Testing Conventions

| Repository | Test Command | Framework |
|------------|--------------|-----------|
| metorial-platform | `bun run test` | Vitest |
| mcp-engine | `make test` | Go testing |

## Code Style

- **TypeScript**: Strict mode, ESLint + Prettier
- **Go**: gofmt, golangci-lint v2 (config: `mcp-engine/.golangci.yml`)
- **Commits**: Conventional commits recommended

**Note**: golangci-lint v2 requires `version: "2"` in config. Key v2 changes:
- `linters-settings` → `linters.settings`
- `issues.exclude-dirs/files` → `linters.exclusions.paths`
- `typecheck` removed (not a linter in v2)
- `gosimple` merged into `staticcheck`

## Changelog Generation

Automated changelog generation via [git-cliff](https://git-cliff.org/) using Conventional Commits.

**Status**: ✅ Fully integrated (2026-02-01)
- CHANGELOG.md files generated for workspace and all projects
- Git hooks auto-configured on workspace entry
- CI workflow includes changelog preview on PRs

```bash
# Preview upcoming changelog
mise run changelog:preview

# Generate workspace changelog
mise run changelog:generate

# Per-repository changelogs
mise run changelog:platform    # metorial-platform
mise run changelog:catalog     # metorial (catalog)
mise run changelog:index       # metorial-index
mise run changelog:engine      # mcp-engine

# Calculate next semantic version
mise run changelog:bump

# Generate release notes
mise run changelog:release
```

**Shell Aliases** (when mise shell integration active):
- `clog` - Preview changelog
- `cgen` - Generate changelog
- `cbump` - Calculate next version

See `changelog_workflow` memory for full documentation.

## Drift Pattern Compliance

Drift Detect provides pattern enforcement and AI context curation.

### Distributed Scan Architecture (2026-02-01)

```
LOCAL DEVELOPMENT (fast feedback)
├─ Pre-commit: drift check --staged (validates staged files)
├─ Pre-push: drift gate (quality gate, no full scan)
└─ Periodic: mise run drift:scan-full (manual, when needed)

CI WORKFLOW (seconds, not hours)
├─ Always uses incremental scan
├─ Validates against committed .drift/ baselines
└─ 15-minute timeout prevents runaway jobs

SCHEDULED JOB (weekly, non-blocking)
├─ Sunday 2 AM UTC
├─ Full pattern rebuild (up to 2 hours)
└─ Generates trend reports and artifacts
```

### Common Commands

```bash
# Incremental scan (fast, use daily)
mise run drift:scan

# Full scan (slow, use periodically - LOCAL ONLY)
mise run drift:scan-full

# Quality gate (CI mode)
mise run drift:gate

# Check project status
mise run drift:status
```

### Git Hooks (Pre-configured)

```bash
# Install hooks (one-time)
git config core.hooksPath .githooks

# Pre-commit: Validates staged files
# Pre-push: Runs quality gate
```

### AI-Assisted Development Workflow

1. Get context: `drift_context` for patterns
2. Find code: Serena `find_symbol`
3. Generate following patterns
4. Edit with Serena tools
5. Validate: `drift_validate_change`

---

**Last Updated**: 2026-02-01
