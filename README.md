# Metorial Workspace

Shared workspace configuration for the Metorial ecosystem - an open source integration platform for agentic AI built on the Model Context Protocol (MCP).

## Quick Start

### 1. Clone this workspace repository

```bash
git clone https://github.com/metorial/metorial-workspace.git metorial
cd metorial
```

### 2. Clone the project repositories

```bash
git clone https://github.com/metorial/metorial.git
git clone https://github.com/metorial/metorial-docs.git
git clone https://github.com/metorial/metorial-index.git
git clone https://github.com/metorial/metorial-platform.git
```

### 3. Install tools with mise

```bash
mise trust
mise install
```

### 4. Open the workspace in VS Code

```bash
code metorial.code-workspace
```

## What's in this repo?

This repository contains **workspace-level configuration only** - the actual project code lives in separate repositories.

| File | Purpose |
|------|---------|
| `mise.toml` | Tool versions and task definitions |
| `.mise/tasks/` | Shared automation scripts |
| `metorial.code-workspace` | VS Code multi-root workspace |
| `CLAUDE.md` | AI assistant context |
| `.markdownlint-cli2.yaml` | Markdown linting rules |

## Project Repositories

| Repository | Description |
|------------|-------------|
| [metorial](https://github.com/metorial/metorial) | MCP Server Catalog - Pre-built MCP servers in Docker |
| [metorial-docs](https://github.com/metorial/metorial-docs) | Documentation site (Mintlify) |
| [metorial-index](https://github.com/metorial/metorial-index) | Curated MCP server registry |
| [metorial-platform](https://github.com/metorial/metorial-platform) | Core platform - backend, frontend, MCP engine |

## Mise Tasks

Run `mise tasks` to see all available commands:

```bash
mise run setup      # Initial workspace setup
mise run dev        # Start development servers
mise run build      # Build entire workspace
mise run test       # Run all tests
mise run status     # Show workspace status
mise run clean      # Clean build artifacts
```

## Tool Versions

Managed by [mise](https://mise.jdx.dev):

| Tool | Version | Purpose |
|------|---------|---------|
| Node.js | 24 LTS | Platform frontend/services |
| Bun | 1.3.x | Runtime and package management |
| Go | 1.24 | MCP engine |
| Yarn | 1.x | Catalog package manager |

## Links

- **Production**: https://metorial.com
- **App**: https://app.metorial.com
- **API Docs**: https://metorial.com/api
- **Mise Docs**: https://mise.jdx.dev
