# Claude Code Skills Integration

This memory documents the Claude Code Skills integration for the Metorial workspace.

## Overview

The workspace implements a 4-Layer Progressive Context Architecture that optimizes AI context loading while ensuring comprehensive coverage for complex development tasks.

## Architecture Layers

### Layer 1: AGENTS.md (Always Loaded)
- ~67 lines of essential workspace context
- Repository structure, package managers, core commands
- AI context tool references
- Code style requirements

### Layer 2: Claude Code Skills (Auto-Triggered)
- Located in `metorial-claude-plugin/`
- Auto-activated by task keywords in skill descriptions
- Each skill ~1.5K words with optional deep-dive references
- Skills reference Layer 3 and 4 tools for deeper context

### Layer 3: Serena Memories (On-Demand)
- Cross-session persistent context
- Called via `read_memory('memory_name')`
- Examples: workspace_overview, development_workflow, mise_integration

### Layer 4: Drift Patterns (Context-Curated)
- 127+ detected patterns from codebase analysis
- Called via `drift_context targeting <path>`
- Provides convention detection and validation

## Skills Inventory

### mcp-server-development
- **Path**: `metorial-claude-plugin/skills/mcp-server-development/`
- **Triggers**: "create MCP server", "build server", "add server to catalog", mentions `metorial/servers/`
- **Purpose**: Guide MCP server development following catalog conventions
- **References**: `server-patterns.md` with patterns from 33+ existing servers

### platform-api-design
- **Path**: `metorial-claude-plugin/skills/platform-api-design/`
- **Triggers**: "add endpoint", "create API", "platform backend", "Hono route", "Prisma model"
- **Purpose**: Guide backend API development with Hono, Prisma, Zod
- **References**: `backend-conventions.md` with architecture patterns

### go-engine-patterns
- **Path**: `metorial-claude-plugin/skills/go-engine-patterns/`
- **Triggers**: "modify engine", "Go code", "gRPC", "mcp-engine", "golangci-lint"
- **Purpose**: Guide Go MCP engine development with golangci-lint v2 compliance
- **References**: `grpc-patterns.md` with streaming, interceptor, connection patterns

## Hooks

The plugin includes convention enforcement hooks in `hooks/hooks.json`:

### PreToolUse Hooks
- **Write|Edit matcher**: Verifies code follows appropriate conventions (Go/TypeScript/MCP)
- **Bash matcher**: Recommends mise tasks for workspace commands

### Stop Hook
- Validates completion checklist: pattern checking, lint compliance, API conventions, Zod validation

## Tool Responsibility Matrix

| Task | Primary Tool | Fallback |
|------|--------------|----------|
| Code navigation | Serena `find_symbol` | Grep/Glob |
| Pattern detection | Drift `drift_context` | Manual review |
| Build/test commands | Mise `mise run` | Direct commands |
| Workflow guidance | Claude Code Skills | AGENTS.md |
| Cross-session memory | Serena `read_memory` | Skills references |

## Plugin Structure

```
metorial-claude-plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── skills/
│   ├── mcp-server-development/
│   │   ├── SKILL.md         # MCP server guidance
│   │   └── references/
│   │       └── server-patterns.md
│   ├── platform-api-design/
│   │   ├── SKILL.md         # Backend API guidance
│   │   └── references/
│   │       └── backend-conventions.md
│   └── go-engine-patterns/
│       ├── SKILL.md         # Go engine guidance
│       └── references/
│           └── grpc-patterns.md
├── hooks/
│   └── hooks.json           # Convention enforcement
└── README.md                # Plugin documentation
```

## Integration with Existing Tools

### With Serena
- Skills reference `read_memory()` for deep context
- Skills suggest `find_symbol` for code navigation
- This memory provides skills documentation

### With Drift
- Skills reference `drift_context` for pattern detection
- Skills suggest `drift_validate_change` for compliance checking
- Pattern files derived from Drift analysis

### With Mise
- Skills reference mise tasks for commands
- Hooks recommend mise over direct commands
- Task inventory in development_workflow memory

## Usage Guidelines

1. **For MCP Server Development**: Skill auto-activates on triggers, references server-patterns.md, suggests Drift for existing patterns
2. **For Platform Backend**: Skill auto-activates on triggers, references backend-conventions.md, suggests Serena for code navigation
3. **For Go Engine**: Skill auto-activates on triggers, references grpc-patterns.md, emphasizes golangci-lint v2 compliance
4. **For Validation**: Stop hook prompts verification checklist before task completion

## Maintenance

- Update skills when new patterns emerge in codebase
- Update references when deep documentation needed
- Update hooks when new conventions established
- Update this memory when architecture changes
