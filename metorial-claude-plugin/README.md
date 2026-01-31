# Metorial Workspace Plugin

Claude Code plugin providing specialized skills, commands, and agents for the Metorial workspace.

## Overview

This plugin integrates with the existing Serena MCP and Drift Detect tooling to provide a comprehensive AI-assisted development experience. It follows the **4-Layer Progressive Context Architecture**:

```
Layer 1: AGENTS.md        → Quick reference, always loaded
Layer 2: Claude Skills    → Auto-triggered procedural workflows
Layer 3: Serena Memories  → Deep context, cross-session persistence
Layer 4: Drift Patterns   → Detected conventions, validation
```

## Skills

### mcp-server-development

**Triggers**: "create MCP server", "build server", "add server to catalog", "implement MCP"

Provides comprehensive guidance for building MCP servers that match existing catalog patterns. References Drift patterns from 33+ existing servers.

### platform-api-design

**Triggers**: "add endpoint", "create API", "platform backend", "Hono route", "Prisma model"

Guides TypeScript backend development following platform conventions (Hono, Prisma, Bun).

### go-engine-patterns

**Triggers**: "modify engine", "Go code", "gRPC", "mcp-engine", "golangci-lint"

Provides Go development guidance for the mcp-engine, including golangci-lint v2 compliance.

## Hooks

### Convention Validation (PreToolUse)

Validates file writes against Metorial conventions:
- MCP servers must follow catalog patterns
- Go code must comply with golangci-lint v2
- Platform code follows TypeScript strict mode

### Task Completion (Stop)

Validates task completion:
- Drift validation for generated code
- Serena memory updates for new patterns

## Integration with Existing Tools

### Serena MCP

Skills reference Serena memories for deep context:
- `read_memory('development_workflow')` for build/test patterns
- `read_memory('workspace_overview')` for project structure
- `find_symbol` for code navigation

### Drift Detect

Skills reference Drift for pattern context:
- `drift_context` for existing patterns
- `drift_validate_change` for compliance
- `drift_code_examples` for real implementations

### Mise

Commands reference Mise for task execution:
- `mise run catalog:build` for server builds
- `mise run engine:lint` for Go linting
- `mise run test` for validation

## Installation

This plugin is automatically discovered when placed in the workspace. Ensure it's registered in `.claude/settings.json`:

```json
{
  "plugins": {
    "metorial-workspace": {
      "path": "./metorial-claude-plugin"
    }
  }
}
```

## Development

Test the plugin locally:

```bash
claude --plugin-dir ./metorial-claude-plugin
```

## License

MIT
