# Metorial Workspace Overview

## Load When
- Session start
- Understanding workspace structure
- Navigating between repositories

---

## Purpose

**Metorial** (YC F25) is an open-source integration platform for agentic AI built on the Model Context Protocol (MCP). This workspace coordinates four related repositories.

## Workspace Structure

```
metorial/                      # Workspace root (this directory)
├── .serena/                   # Serena project configuration
├── mise.toml                  # Unified tooling configuration
├── CLAUDE.md                  # AI assistant instructions
│
├── metorial/                  # MCP Server Catalog
│   ├── servers/               # 33+ custom MCP server implementations
│   ├── packages/              # Shared packages (SDK, manifest, etc.)
│   ├── catalog/               # Third-party server index (408+)
│   └── scripts/               # Development automation
│
├── metorial-platform/         # Core Platform
│   ├── src/backend/           # API server
│   ├── src/frontend/          # React dashboard
│   ├── src/mcp-engine/        # Go-based MCP connection engine
│   ├── src/services/          # Microservices
│   └── src/packages/          # Shared platform packages
│
├── metorial-index/            # Server Registry
│   └── scripts/               # Indexing and sync tools
│
└── metorial-docs/             # Documentation Site
    └── *.mdx                  # Mintlify documentation
```

## Technology Stack

| Component | Repository | Tech |
|-----------|------------|------|
| MCP Servers | metorial | TypeScript, Bun, Docker |
| Platform Backend | metorial-platform | TypeScript, Bun, Turbo |
| MCP Engine | metorial-platform/src/mcp-engine | Go 1.24 |
| Frontend | metorial-platform | React, TypeScript |
| Documentation | metorial-docs | Mintlify, MDX |
| Server Index | metorial-index | TypeScript, Bun |

## Quick Navigation

| To find... | Look in... |
|------------|------------|
| MCP server implementations | `metorial/servers/<name>/` |
| SDK for building servers | `metorial/packages/sdk/` |
| Platform API endpoints | `metorial-platform/src/backend/` |
| Go MCP engine | `metorial-platform/src/mcp-engine/` |
| Documentation content | `metorial-docs/*.mdx` |
| Build/task commands | `mise.toml` at workspace root |

---

**Last Updated**: 2026-01-30
