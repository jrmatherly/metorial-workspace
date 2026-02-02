# Git Hooks for Metorial Workspace

Git hooks are managed by [mise-hk](https://github.com/jdx/hk) via `hk.pkl`.

## Current Setup

Hooks are defined in `hk.pkl` and installed to `.git/hooks/`:

```bash
# Install hooks (with mise integration)
hk install --mise

# Check for issues manually
hk check

# Auto-fix issues
hk fix

# Run specific hook
hk run pre-commit
```

### Hooks Configured

| Hook | Purpose |
|------|---------|
| `pre-commit` | Drift staged file checks |
| `commit-msg` | Conventional commits validation |
| `pre-push` | Drift quality gate |

### Claude Code Compatibility

Hooks run silently (output redirected to `/dev/null`) to prevent Claude Code crashes from hk's streaming output. Run `hk check` manually to see issues before committing.

## Conventional Commits

All commit messages must follow [Conventional Commits](https://www.conventionalcommits.org/) format:

- **Format:** `type(scope): description`
- **Valid types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`
- **Breaking changes:** Add `!` after type/scope (e.g., `feat(api)!: breaking change`)
- **Bypass:** `git commit --no-verify` (not recommended)

## Requirements

- **mise** must be installed and active
- **hk** and **pkl** installed via mise (`mise install`)
- **Drift** for pattern checks (`mise install`)

## Troubleshooting

```bash
# Verify hk is working
hk validate
hk check

# Run with verbose output
HK_LOG=debug hk check

# Check Drift separately
drift check --staged --verbose
drift gate --verbose
```
