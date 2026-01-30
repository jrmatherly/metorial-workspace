# Metorial Ecosystem

This workspace contains the Metorial ecosystem - an open source integration platform for agentic AI built on the Model Context Protocol (MCP).

## Repository Overview

| Repository | Purpose | Tech Stack | Package Manager |
|------------|---------|------------|-----------------|
| **metorial** | MCP Server Catalog - Pre-built MCP servers in Docker containers | TypeScript, Bun | Yarn (workspaces) |
| **metorial-docs** | Documentation site (Mintlify) | MDX | Mintlify CLI |
| **metorial-index** | Curated MCP server index/registry | TypeScript, Bun | Bun (workspaces) |
| **metorial-platform** | Core platform - backend, frontend, MCP engine | TypeScript, Go, React | Bun + Turbo |

## Repository Relationships

```mermaid
┌─────────────────────────────────────────────────────────────┐
│                    metorial-platform                        │
│  (Core Platform: API, Dashboard, MCP Engine)                │
│                                                             │
│  ┌─────────┐ ┌──────────┐ ┌───────────┐ ┌─────────────────┐ │
│  │ backend │ │ frontend │ │mcp-engine │ │ services/modules│ │
│  └─────────┘ └──────────┘ └───────────┘ └─────────────────┘ │
└─────────────────────────────────────────────────────────────┘
         │                          │
         │ uses                     │ runs
         ▼                          ▼
┌─────────────────┐        ┌──────────────────┐
│    metorial     │◄───────│  metorial-index  │
│  (MCP Catalog)  │ indexes│ (Server Registry)│
│                 │        │                  │
│ servers/*       │        │ catalog/*        │
│ packages/*      │        │                  │
└─────────────────┘        └──────────────────┘
         │
         │ documented in
         ▼
┌─────────────────┐
│  metorial-docs  │
│ (Documentation) │
│                 │
│ *.mdx files     │
└─────────────────┘
```

## Cross-Repo Patterns

### Shared Conventions

- **Package naming**: `@metorial/*` or `@metorial-<repo>/root`
- **Runtime**: Bun for TypeScript, Go for performance-critical engine
- **Build tool**: Turbo (platform), Yarn workspaces (catalog), Bun workspaces (index)

### Key Dependencies

- **metorial-platform** → references **metorial** for MCP server definitions
- **metorial-index** → indexes servers from **metorial** catalog
- **metorial-docs** → documents all components, references platform APIs

## Mise-en-Place (Project Management)

This workspace uses [mise](https://mise.jdx.dev) for unified tool version management and task orchestration.

### Quick Start

```bash
# Initial setup (installs tools + dependencies)
mise run setup

# List all available tasks
mise tasks

# Check workspace status
mise run status
```

### Tool Versions (managed by mise)

| Tool | Version | Purpose |
|------|---------|---------|
| Node.js | 24 LTS | Platform frontend/services |
| Bun | 1.3.x | Runtime and package management |
| Go | 1.24 | MCP engine |
| Yarn | 1.x | Catalog package manager |

### Common Tasks

```bash
# Development
mise run dev                    # Start all dev servers
mise run platform:dev           # Platform only
mise run docs:dev               # Documentation server
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

# Database
mise run platform:prisma:generate  # Generate Prisma client
mise run platform:prisma:push      # Push schema changes

# Utilities
mise run status                 # Workspace status
mise run clean                  # Clean all artifacts
```

### MCP Server Integration

Mise provides an experimental MCP server for AI assistants:

```bash
# Query tools, tasks, and environment via MCP protocol
mise mcp
```

Resources available: `mise://tools`, `mise://tasks`, `mise://env`, `mise://config`

### Serena Integration

This workspace uses [Serena](https://github.com/oraios/serena) for semantic code understanding via Language Server Protocol (LSP). Serena provides IDE-like tools for finding symbols, references, and precise code editing.

**Configuration**: `.serena/project.yml`
- Languages: TypeScript + Go
- Context: `claude-code` (optimized for Claude Code)

**Memories**: `.serena/memories/`
- `workspace_overview.md` - Workspace structure and navigation
- `mise_integration.md` - Mise tooling reference
- `development_workflow.md` - Development patterns

**Key Serena tools**:
- `find_symbol` - Find classes, functions, methods by name
- `find_referencing_symbols` - Find all usages of a symbol
- `get_symbols_overview` - Get file structure without reading full content
- `read_memory` / `write_memory` - Persistent project knowledge

## Legacy Commands (Direct)

### metorial (MCP Catalog)

```bash
cd metorial
bun run add-server      # Add new MCP server
bun run build           # Build all servers
bun run check-versions  # Verify version consistency
```

### metorial-docs (Documentation)

```bash
cd metorial-docs
npm i -g mint          # Install Mintlify CLI (one-time)
mint dev               # Local preview at localhost:3000
```

### metorial-index (Server Index)

```bash
cd metorial-index
# Uses Bun workspaces - check scripts/* for tooling
```

### metorial-platform (Platform)

```bash
cd metorial-platform
bun install            # Install dependencies
bun run build          # Build all packages (Turbo)
bun run dev            # Development mode
bun run test           # Run tests
bun run prisma:generate # Generate Prisma client
```

### MCP Engine (Go - Makefile)

```bash
cd metorial-platform/src/mcp-engine
make build-unified     # Build main binary
make dev               # Hot reload with air
make test              # Run tests
make lint              # golangci-lint
make proto             # Generate protobuf
```

## Architecture Notes

### metorial-platform Structure

- `src/backend/` - API server and business logic
- `src/frontend/` - React dashboard
- `src/mcp-engine/` - Go-based MCP connection engine (high performance)
- `src/services/` - Microservices
- `src/modules/` - Shared modules
- `src/packages/` - Shared packages
- `src/runner/` - MCP server runner
- `vendor/` - Vendored dependencies

### Tech Stack

- **Languages**: TypeScript (primary), Go (MCP engine)
- **Frontend**: React
- **Databases**: PostgreSQL, MongoDB (logs), Redis (caching)
- **Containerization**: Docker for MCP servers
- **Build**: Turbo, Bun, Yarn

## Working Across Repos

### Claude Code Multi-Directory

```bash
# From any repo, add others to context
/add-dir ../metorial-platform
/add-dir ../metorial-docs
```

### Common Tasks

- **Adding a new MCP server**: Start in `metorial/`, update `metorial-index/`, document in `metorial-docs/`
- **Platform features**: Work in `metorial-platform/src/`
- **API documentation**: Update `metorial-docs/api-*.mdx` files

## Git Workflow

Each repository is a separate Git repo with its own history. When making cross-repo changes:

1. Make changes in dependency order (metorial → metorial-index → metorial-platform)
2. Commit and push each repo independently
3. Update any version references as needed

## CI/CD Integration

### GitHub Actions with Mise

```yaml
- uses: jdx/mise-action@v3
  with:
    version: latest
    install: true
    cache: true
    experimental: true

- run: mise run build
- run: mise run test
```

## Links

- **Production**: https://metorial.com
- **App**: https://app.metorial.com
- **API Docs**: https://metorial.com/api
- **MCP Protocol**: https://modelcontextprotocol.io
- **Mise Documentation**: https://mise.jdx.dev
