# Metorial Workspace

Shared workspace configuration for the Metorial ecosystem - an open source integration platform for agentic AI built on the Model Context Protocol (MCP).

## Quick Start

```bash
# Clone workspace and initialize everything
git clone https://github.com/jrmatherly/metorial-workspace.git metorial
cd metorial
mise trust
mise run init

# Open in VS Code
code metorial.code-workspace
```

That's it! The `mise run init` command will:
1. Clone all project repositories
2. Install mise-managed tools (node, bun, go, yarn)
3. Install platform dependencies

### Manual Setup (Alternative)

If you prefer to clone repositories manually:

```bash
git clone https://github.com/jrmatherly/metorial-workspace.git metorial
cd metorial
git clone https://github.com/jrmatherly/metorial.git
git clone https://github.com/jrmatherly/metorial-docs.git
git clone https://github.com/jrmatherly/metorial-index.git
git clone https://github.com/jrmatherly/metorial-platform.git
mise trust && mise install
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
| [metorial](https://github.com/jrmatherly/metorial) | MCP Server Catalog - Pre-built MCP servers in Docker |
| [metorial-docs](https://github.com/jrmatherly/metorial-docs) | Documentation site (Mintlify) |
| [metorial-index](https://github.com/jrmatherly/metorial-index) | Curated MCP server registry |
| [metorial-platform](https://github.com/jrmatherly/metorial-platform) | Core platform - backend, frontend, MCP engine |

## Mise Tasks

Run `mise tasks` to see all available commands:

```bash
# Onboarding
mise run init         # Complete workspace initialization (clone, install, setup)
mise run clone-repos  # Clone project repositories only

# Development
mise run dev          # Start development servers
mise run build        # Build entire workspace
mise run test         # Run all tests

# Utilities
mise run setup        # Install tools and dependencies
mise run status       # Show workspace status
mise run clean        # Clean build artifacts
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
