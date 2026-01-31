# AGENTS.md Architecture

## Load When
- Understanding AI context hierarchy
- Troubleshooting agent context issues
- Modifying AGENTS.md files

---

## Overview

This workspace implements the [AGENTS.md open standard](https://agents.md/) for AI agent context.

## Context Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│ Layer 1: AGENTS.md (Root)           ~67 lines, EVERY REQUEST    │
│   Essential briefing: project desc, commands, tool references   │
├─────────────────────────────────────────────────────────────────┤
│ Layer 2: Subdirectory AGENTS.md     ~30 lines, WHEN IN SCOPE    │
│   metorial-platform/AGENTS.md       Platform build/test         │
│   metorial-platform/src/mcp-engine/AGENTS.md  Go engine         │
│   metorial/AGENTS.md                Catalog servers             │
├─────────────────────────────────────────────────────────────────┤
│ Layer 3: Serena Memories            ON-DEMAND via read_memory   │
│   workspace_overview, development_workflow, mise_integration    │
├─────────────────────────────────────────────────────────────────┤
│ Layer 4: Drift Context              ON-DEMAND via drift:context │
│   Code patterns, conventions, call graphs                       │
└─────────────────────────────────────────────────────────────────┘
```

## Tool Interoperability

| Tool | Context Type | Token Impact | Best For |
|------|--------------|--------------|----------|
| **AGENTS.md** | Static | HIGH (every request) | Essential routing |
| **Serena** | On-demand | LOW | Deep workflows, architecture |
| **Drift** | Generated | LOW | Code patterns |
| **Mise** | Reference | MINIMAL | Task execution |

## File Locations

| File | Lines | Purpose |
|------|-------|---------|
| `AGENTS.md` | 67 | Root context (symlinked from CLAUDE.md) |
| `metorial-platform/AGENTS.md` | 37 | Platform-specific |
| `metorial-platform/src/mcp-engine/AGENTS.md` | 45 | Go engine |
| `metorial/AGENTS.md` | 40 | Catalog servers |

## Best Practices

1. **Keep root minimal** - ≤150 lines, ideally ≤100
2. **Use progressive disclosure** - Link to Serena memories for deep context
3. **No file paths** - They go stale; describe capabilities instead
4. **Update with code** - Treat AGENTS.md like code in PRs

## CLAUDE.md Compatibility

CLAUDE.md is a symlink to AGENTS.md:
```bash
CLAUDE.md -> AGENTS.md
```

This ensures Claude Code reads the same content as other AGENTS.md-compatible tools
(Cursor, Aider, Gemini CLI, Codex, etc.)

## Legacy Reference

Original CLAUDE.md archived at `CLAUDE.md.legacy` (353 lines).
Content migrated to:
- Root AGENTS.md (essential commands)
- Serena memories (detailed workflows)
- Subdirectory AGENTS.md files (repo-specific)

---

**Last Updated**: 2026-01-31
