# Changelog Workflow

## Load When
- Creating commits or releases
- Generating changelogs
- Reviewing commit history
- Automating release notes

---

## Overview

This workspace uses [git-cliff](https://git-cliff.org/) for automated changelog generation following the [Conventional Commits](https://www.conventionalcommits.org/) specification.

## Commit Message Format

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types

| Type | Description | SemVer |
|------|-------------|--------|
| `feat` | New feature | Minor |
| `fix` | Bug fix | Patch |
| `docs` | Documentation changes | - |
| `style` | Code style (formatting, etc.) | - |
| `refactor` | Code refactoring | - |
| `perf` | Performance improvements | Patch |
| `test` | Test additions/modifications | - |
| `build` | Build system changes | - |
| `ci` | CI/CD changes | - |
| `chore` | Maintenance tasks | - |
| `revert` | Revert previous commit | - |

### Breaking Changes

- Add `!` after type/scope: `feat!: breaking change`
- Or add `BREAKING CHANGE:` in footer

### Examples

```bash
# Feature with scope
feat(auth): add OAuth2 support for GitHub

# Bug fix
fix: resolve null pointer in session handler

# Breaking change
feat(api)!: change response format to JSON:API

# With body and footer
fix(engine): handle connection timeout gracefully

The MCP engine now properly handles timeout scenarios
by implementing exponential backoff retry logic.

Closes #123
```

## Common Tasks

```bash
# Preview upcoming changelog (dry run)
mise run changelog:preview

# Generate full workspace changelog
mise run changelog:generate

# Per-repository changelogs
mise run changelog:platform    # metorial-platform
mise run changelog:catalog     # metorial (catalog)
mise run changelog:index       # metorial-index
mise run changelog:engine      # mcp-engine

# Generate all changelogs at once
mise run changelog:all

# Calculate next semantic version
mise run changelog:bump

# Generate release notes for latest tag
mise run changelog:release

# Validate commit format
mise run changelog:validate
```

### Shell Aliases

When mise shell integration is active:

| Alias | Command |
|-------|---------|
| `clog` | `mise run changelog:preview` |
| `cgen` | `mise run changelog:generate` |
| `cbump` | `mise run changelog:bump` |

## Git Hooks Integration

Git hooks are **automatically configured** when entering the workspace (via mise `enter` hook).

### Manual Installation

```bash
mise run git:hooks:install    # Enable hooks
mise run git:hooks:uninstall  # Disable hooks
```

### commit-msg Hook

Validates conventional commit format before allowing commits:

```bash
# Valid examples
feat(auth): add OAuth2 support
fix: resolve null pointer exception
docs(readme)!: breaking change to docs

# Invalid - will be rejected
"Updated some stuff"
```

## Configuration

| File | Purpose |
|------|---------|
| `cliff.toml` | Main git-cliff configuration |
| `mise.toml` | Task definitions (changelog:* namespace) |

### Key Configuration Options

```toml
# cliff.toml highlights
[git]
conventional_commits = true      # Parse conventional format
filter_unconventional = true     # Exclude non-conventional
protect_breaking_commits = true  # Never filter breaking changes

[remote.github]
owner = "jrmatherly"            # GitHub org/user
repo = "metorial-platform"      # Default repo (override per-task)
```

## Integration Points

### Mise Task Orchestration
All changelog operations are orchestrated via mise tasks in the `changelog:*` namespace.

### Drift Detect
Commit message quality can be validated via Drift patterns for conventional commit compliance.

### GitHub Actions
- PR previews: Generate changelog preview in pull requests
- Release automation: Auto-generate release notes on tag push
- Commit validation: Enforce conventional commits in CI

### Serena
This memory provides AI context for changelog-related tasks.

## Workflow Examples

### Daily Development
```bash
# Before committing, preview what would be in changelog
mise run changelog:preview

# After tagging a release
mise run changelog:generate
git add CHANGELOG.md
git commit -m "chore(release): update changelog"
```

### Release Process
```bash
# 1. Check what version bump is needed
mise run changelog:bump

# 2. Generate release notes
mise run changelog:release

# 3. Update full changelog
mise run changelog:generate

# 4. Commit and tag
git add CHANGELOG.md RELEASE_NOTES.md
git commit -m "chore(release): prepare v1.2.0"
git tag v1.2.0
```

### Per-Repository Changelogs
```bash
# Generate changelog for specific project
mise run changelog:platform

# Or generate all at once
mise run changelog:all
```

---

**Last Updated**: 2026-01-31
