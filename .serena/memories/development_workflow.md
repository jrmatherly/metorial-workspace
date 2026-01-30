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
- **Go**: gofmt, golangci-lint
- **Commits**: Conventional commits recommended

---

**Last Updated**: 2026-01-30
