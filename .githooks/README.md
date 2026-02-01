# Git Hooks for Drift Integration

This directory contains Git hooks that integrate Drift quality checks into the development workflow.

## Installation

To enable these hooks, run:

```bash
git config core.hooksPath .githooks
```

Or use the mise task:

```bash
mise run drift:hooks:install
```

## Hooks

### pre-commit

Runs `drift check --staged` on staged TypeScript/JavaScript files before each commit.

- **Fails on:** Errors only (warnings are allowed)
- **Bypass:** `git commit --no-verify` (not recommended)

### pre-push

Runs quality gate checks before pushing to remote.

- **Protected branches (main/master/develop):** Full gate, fails on warnings
- **Feature branches:** Lighter checks, fails on errors only
- **Bypass:** `git push --no-verify` (not recommended)

## Disabling Hooks

To disable hooks temporarily:

```bash
git config --unset core.hooksPath
```

To re-enable:

```bash
git config core.hooksPath .githooks
```

## Requirements

- Drift must be installed (`mise install npm:driftdetect`)
- `.drift/` directory must be initialized (`drift init`)

## Troubleshooting

If hooks fail unexpectedly:

1. Run `drift check --staged --verbose` to see detailed output
2. Run `drift gate --verbose` to see gate details
3. Check that Drift is properly installed: `drift --version`

If Drift is not installed, hooks will skip silently with a warning.
