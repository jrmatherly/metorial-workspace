# Drift Detect Integration Analysis (VALIDATED)

**Status**: Configuration validated against official documentation
**Last Validated**: 2026-01-30

## Load When
- Planning Drift integration
- Understanding AI tooling architecture
- Configuring MCP servers
- Setting up quality gates

---

## Executive Summary

**Drift Detect** is a codebase intelligence platform that learns patterns from your code and provides that knowledge to AI agents. It complements the existing Serena integration by adding:

- **Pattern Detection**: 400+ detectors across 15 categories
- **Convention Enforcement**: AI generates code that matches YOUR patterns
- **Call Graph Analysis**: Data flow tracking, impact analysis
- **Security Analysis**: Sensitive data tracking and boundaries
- **Quality Gates**: CI/CD integration with pattern compliance

## What Drift Detect Is

### Core Capabilities

| Feature | Description | Performance |
|---------|-------------|-------------|
| Pattern Detection | 400+ detectors for API, auth, errors, etc. | 127 pattern types typical |
| Call Graph | Function call mapping with data flow | 10K files in 2.3s (Rust core) |
| Security Boundaries | Sensitive data tracking | AST-first analysis |
| Test Topology | Test-to-code mapping | 30+ test frameworks |
| Module Coupling | Dependency cycle detection | O(V+E) Tarjan's algorithm |
| MCP Server | 50 tools in 7-layer architecture | Token-budget optimized |

### Language Support

| Language | Framework Support |
|----------|-------------------|
| TypeScript/JavaScript | Express, NestJS, Next.js, React |
| Go | Gin, Echo, Chi, Fiber |
| Python | FastAPI, Django, Flask |
| Java | Spring Boot, JAX-RS |
| C# | ASP.NET Core, WPF |
| Rust | Actix, Axum, Rocket |
| PHP | Laravel, Symfony |
| C/C++ | Memory patterns, templates |

### 7-Layer MCP Architecture

```
Layer 1: ORCHESTRATION    → drift_context, drift_package_context
Layer 2: DISCOVERY        → drift_status, drift_capabilities
Layer 3: SURGICAL         → drift_signature, drift_callers, drift_imports (12 tools)
Layer 4: EXPLORATION      → drift_patterns_list, drift_security_summary
Layer 5: DETAIL           → drift_pattern_get, drift_code_examples
Layer 6: ANALYSIS         → drift_test_topology, drift_coupling
Layer 7: GENERATION       → drift_suggest_changes, drift_validate_change
```

## How Drift Complements Serena

### Division of Responsibilities

| Task | Best Tool | Reason |
|------|-----------|--------|
| Find symbol definition | **Serena** | LSP provides exact location |
| Find all usages | **Serena** | LSP references are precise |
| Edit/replace symbol body | **Serena** | Semantic editing capability |
| Rename across codebase | **Serena** | LSP rename refactoring |
| Understand codebase patterns | **Drift** | Pattern detection engine |
| Generate code that fits conventions | **Drift** | `drift_context` curates patterns |
| Validate AI-generated code | **Drift** | `drift_validate_change` |
| Cross-file impact analysis | **Drift** | Call graph + impact analysis |
| Security data flow tracking | **Drift** | `drift_reachability` |
| Test coverage mapping | **Drift** | `drift_test_topology` |

### Recommended Workflow

```
1. drift_context → Understand patterns and get curated context
2. Serena find_symbol → Navigate to target code
3. Serena read_memory → Check project conventions
4. Generate/edit code using patterns from Drift
5. Serena replace_symbol_body → Apply changes
6. drift_validate_change → Verify pattern compliance
```

### No Conflicts

Serena and Drift operate on different abstraction levels:
- **Serena**: LSP-level operations (symbols, references, precise edits)
- **Drift**: Pattern-level operations (conventions, data flow, AI context)

They share no overlapping functionality and can coexist as separate MCP servers.

## Mise Integration Plan

### Recommended Tasks

```toml
# .mise.toml additions

# ============================================================================
# DRIFT TASKS - Codebase Intelligence
# ============================================================================
[tasks."drift:init"]
description = "Initialize Drift in a repository"
run = "drift init"

[tasks."drift:scan"]
description = "Scan repository for patterns"
run = "drift scan"

[tasks."drift:scan-incremental"]
description = "Incremental pattern scan"
run = "drift scan --incremental"

[tasks."drift:status"]
description = "Show pattern status"
run = "drift status --detailed"

[tasks."drift:check"]
description = "Check staged changes for pattern compliance"
run = "drift check --staged"

[tasks."drift:gate"]
description = "Run quality gate (CI)"
run = "drift gate --policy strict --format github"

[tasks."drift:approve"]
description = "Approve discovered patterns"
run = "drift approve --min-confidence 0.9"
usage = 'arg "[pattern-id]" help="Optional pattern ID to approve"'

[tasks."drift:callgraph"]
description = "Build call graph"
run = "drift callgraph build"

[tasks."drift:test-topology"]
description = "Build test topology"
run = "drift test-topology build"
```

### Workspace-Level Tasks

```toml
[tasks."drift:scan-all"]
description = "Scan all repositories for patterns"
depends = ["drift:scan-platform", "drift:scan-catalog", "drift:scan-index"]
run = "echo 'All repositories scanned'"

[tasks."drift:scan-platform"]
dir = "metorial-platform"
run = "drift scan --incremental"

[tasks."drift:scan-catalog"]
dir = "metorial"
run = "drift scan --incremental"

[tasks."drift:scan-index"]
dir = "metorial-index"
run = "drift scan --incremental"
```

## Monorepo Configuration

### Per-Repository `.drift/config.json`

```json
{
  "version": "2.0.0",
  "project": {
    "name": "metorial-platform",
    "description": "Core platform - API, dashboard, MCP engine"
  },
  "ignore": [
    "node_modules/**",
    "dist/**",
    "**/*.test.ts",
    "**/*.spec.ts"
  ],
  "learning": {
    "autoApproveThreshold": 0.90,
    "minOccurrences": 3
  },
  "features": {
    "callGraph": true,
    "boundaries": true,
    "contracts": true
  },
  "mcp": {
    "tools": {
      "languages": ["typescript", "go"],
      "exclude": ["drift_wpf", "drift_php", "drift_java"]
    }
  }
}
```

### Workspace-Level Configuration

Create a root-level `.drift/config.json` for workspace-wide settings:

```json
{
  "monorepo": {
    "enabled": true,
    "root": ".",
    "packages": {
      "metorial": {
        "path": "metorial",
        "language": "typescript",
        "categories": ["api", "data-access"]
      },
      "metorial-platform": {
        "path": "metorial-platform",
        "language": "typescript",
        "framework": "react"
      },
      "mcp-engine": {
        "path": "metorial-platform/src/mcp-engine",
        "language": "go"
      }
    }
  }
}
```

## MCP Server Configuration

### Claude Code Settings (`.claude/settings.json`)

```json
{
  "mcpServers": {
    "drift": {
      "command": "driftdetect-mcp",
      "env": {
        "DRIFT_PROJECT_PATH": "/Users/jason/dev/new-ai/metorial"
      }
    }
  }
}
```

### Coexistence with Serena

Both MCP servers can run simultaneously:
- **Serena**: Provides semantic code editing tools
- **Drift**: Provides pattern context and validation tools

No conflicts expected as they serve different purposes.

## Key Benefits for Metorial

### For AI-Assisted Development

1. **Pattern Learning**: Drift learns MCP server patterns from the catalog
2. **Convention Enforcement**: Generated servers match existing patterns
3. **Cross-Repo Impact**: Understand how shared packages affect consumers
4. **Security Analysis**: Track sensitive data across the platform

### For CI/CD

1. **Quality Gates**: Enforce pattern compliance in PRs
2. **Regression Detection**: Catch pattern drift over time
3. **Test Coverage**: Map tests to code for minimum test sets

### For Onboarding

1. **Pattern Documentation**: Auto-generated pattern descriptions
2. **Code Examples**: Real examples from the codebase
3. **Skills**: 71 implementation guides for common patterns

## Installation Status (ENHANCED 2026-02-01)

Drift is now configured using **Multi-Project Registration** with file-based mise tasks.

### Cortex V2 Memory System (INITIALIZED 2026-02-01)

**Status**: ✅ ACTIVE
**Database**: `.drift/memory/cortex.db`
**Health Score**: 100/100

#### Initial Memory Inventory

| Type | Count | Examples |
|------|-------|----------|
| `decision_context` | 4 | Project identity, Go engine choice, TypeScript strict, Hono routing |
| `tribal` | 4 | Security (bcrypt, auth middleware), architecture, data validation |
| `procedural` | 2 | Deployment workflow, MCP server creation |
| `pattern_rationale` | 1 | Repository pattern for testability |
| `code_smell` | 1 | Avoid `any` type in TypeScript |

#### Cortex Mise Tasks

```bash
# Core operations
mise run cortex:status       # View memory statistics
mise run cortex:health       # Health report with recommendations
mise run cortex:list         # List all memories
mise run cortex:search       # Semantic search

# Knowledge management
mise run cortex:add-tribal   # Add tribal knowledge
mise run cortex:learn        # Learn from corrections
mise run cortex:why          # Get causal narrative
mise run cortex:feedback     # Adjust memory confidence

# Maintenance
mise run cortex:consolidate  # Merge episodic → semantic
mise run cortex:validate     # Validate and heal issues
mise run cortex:export       # Backup to JSON
mise run cortex:import       # Restore from JSON

# Context loading
mise run cortex:predict      # Predictive retrieval
mise run cortex:warnings     # Show active warnings
```

#### Integration with Serena

Cortex and Serena operate at different abstraction levels:
- **Serena**: LSP-level (symbols, references, precise edits)
- **Cortex**: Knowledge-level (learning, decay, intent-aware retrieval)

Synergy pattern:
1. Serena memories → Architectural context (deep, curated)
2. Cortex memories → Dynamic knowledge (learning, decay, validation)
3. No duplication - complementary purposes

### Registered Projects

| Project Name | Path | Language |
|--------------|------|----------|
| MCP Catalog | metorial/ | TypeScript |
| Platform | metorial-platform/ | TypeScript |
| MCP Engine | metorial-platform/src/mcp-engine/ | Go |
| Index Registry | metorial-index/ | TypeScript |

### New Capabilities Added (2026-01-31)

#### 1. Security Boundary Rules
- Location: `.drift/boundaries/rules.json`
- Enforces credential access restricted to auth modules
- Enforces PII access restricted to user services
- Prevents sensitive data in logging
- Prevents hardcoded database URLs outside config

#### 2. Git Hooks Integration
- Location: `.githooks/pre-commit`, `.githooks/pre-push`
- Pre-commit: `drift check --staged` on TS/JS files
- Pre-push: `drift gate` (strict for main, relaxed for features)
- Install: `mise run drift:hooks:install`

#### 3. Watch Mode Tasks
```bash
mise run drift:watch              # Real-time pattern detection
mise run drift:watch-verbose      # Detailed output
mise run drift:watch-api          # API/auth/error patterns only
mise run drift:watch-security     # Security-focused patterns
mise run drift:watch-context      # Auto-update AI context file
```

#### 4. Security & Boundaries Tasks
```bash
mise run drift:boundaries         # Show data access overview
mise run drift:boundaries-check   # Check for violations
mise run drift:boundaries-sensitive # Show sensitive field access
```

#### 5. Trends & Decision Mining
```bash
mise run drift:trends             # 7-day pattern trends
mise run drift:trends-30d         # 30-day trends
mise run drift:decisions          # Show mined decisions
mise run drift:decisions-generate # Generate ADRs from git
```

#### 6. Constants & Secrets Analysis
```bash
mise run drift:constants          # Analyze hardcoded values
mise run drift:constants-secrets  # Find potential secrets
mise run drift:constants-urls     # Find hardcoded URLs
```

#### 7. Impact Analysis
```bash
mise run drift:impact             # Impact of staged changes
mise run drift:impact-file <path> # Impact of specific file
```

#### 8. Enhanced CI Integration
- Proper `.drift/` caching between runs
- Incremental scans on PRs, full scans on main
- SARIF output for GitHub Code Scanning
- Security boundary checks in CI
- Trend analysis on main branch pushes

#### 9. Custom Quality Gate Policy
- Location: `.drift/quality-gates/policies/metorial.json`
- Pattern compliance threshold: 75%
- Security boundary enforcement
- Impact simulation with max blast radius
- Secrets detection blocking

### File-Based Task Architecture (2026-02-01)

All drift tasks are now defined as executable shell scripts in `.mise/tasks/drift/`:

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

### Workflow Architecture (Optimized 2026-02-01)

The Drift integration follows a **baseline-first architecture** to minimize CI friction:

```
┌─────────────────────────────────────────────────────────────────┐
│                     DRIFT BASELINE ARCHITECTURE                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LOCAL DEVELOPMENT                                               │
│  ├─ Pre-commit: drift check --staged (validates staged files)   │
│  ├─ Pre-push: drift gate (quality gate, no full scan)           │
│  └─ Periodic: mise run drift:scan-full (manual, when needed)    │
│                                                                  │
│  CI WORKFLOW (Fast, seconds - NO SCANNING)                       │
│  ├─ Uses committed baselines only (NO drift scan)               │
│  ├─ Runs drift gate against approved patterns                   │
│  ├─ 5-minute timeout (quality checks are fast)                  │
│  └─ Baselines come from git clone, no separate cache needed     │
│                                                                  │
│  WHY CI DOESN'T SCAN:                                            │
│  ├─ Scanning = pattern discovery (expensive, 30+ minutes)       │
│  ├─ Gate = validation against approved patterns (fast, seconds) │
│  ├─ Baselines should be updated locally and committed           │
│  └─ Regenerating baselines in CI defeats the purpose            │
│                                                                  │
│  SCHEDULED JOB (Weekly, non-blocking)                            │
│  ├─ Runs every Sunday 2 AM UTC                                  │
│  ├─ Full pattern rebuild (up to 2 hours allowed)                │
│  └─ Generates trend reports and artifacts                        │
│                                                                  │
│  WHAT'S COMMITTED TO GIT (baselines):                            │
│  ├─ .drift/config.json (project settings)                       │
│  ├─ .drift/patterns/approved/*.json (approved patterns)         │
│  ├─ .drift/indexes/*.json (file hash indexes)                   │
│  ├─ .drift/views/*.json (status caches)                         │
│  └─ .drift/manifest.json (project metadata)                     │
│                                                                  │
│  WHAT'S GITIGNORED (regeneratable):                              │
│  ├─ .drift/lake/ (call graph DB)                                │
│  ├─ .drift/cache/ (temporary)                                   │
│  ├─ .drift/patterns/discovered/ (pending review)                │
│  └─ .drift/**/.backups/ (backup files)                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Commands

```bash
# Scan all Metorial projects (incremental)
mise run drift:scan

# Full scan (ignores cache) - LOCAL ONLY, not for CI
mise run drift:scan-full

# Check project status
mise run drift:status

# Generate AI context
mise run drift:context

# Watch mode during development
mise run drift:watch

# Open web dashboard
mise run drift:dashboard

# Security boundary check
mise run drift:boundaries-check

# Quality gate (CI mode)
mise run drift:gate

# Approve high-confidence patterns
mise run drift:approve-all
```

### Architecture

Each sub-repo has its own `.drift/` directory with independent:
- Pattern tracking
- Health scoring
- Configuration

Workspace-level configuration includes:
- `.drift/boundaries/rules.json` - Security rules
- `.drift/quality-gates/policies/metorial.json` - Custom gate policy
- `.githooks/` - Git hook scripts
- `.github/skills/` - Agent Skills for AI discovery

### Agent Skills (17 installed)

Skills are production-ready implementation guides that AI agents auto-discover in `.github/skills/`.

| Category | Skills |
|----------|--------|
| Resilience | circuit-breaker, retry-fallback, graceful-shutdown |
| Security | jwt-auth, middleware-protection, audit-logging, webhook-security |
| API | rate-limiting, request-validation, api-versioning, idempotency |
| Operations | health-checks, logging-observability |
| Errors | error-handling |
| Foundations | typescript-strict |
| Frontend & UI | better-icons, vercel-react-best-practices |

**Note**: The `drift skills install` CLI command searches an empty remote registry. Skills are installed manually from `.local_docs/drift-docs/skills/` (75 available) to `.github/skills/`.

Usage examples:
- "Add circuit breaker to API client"
- "Find an icon for navigation"
- "Optimize this React component"

### Shell Aliases

```bash
dscan    # mise run drift:scan
dstatus  # mise run drift:status
dcheck   # mise run drift:check
dgate    # mise run drift:gate
dwatch   # mise run drift:watch
dbound   # mise run drift:boundaries
dtrend   # mise run drift:trends
```

---

**Last Updated**: 2026-02-01
