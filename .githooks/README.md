# Git Hooks for Metorial Workspace

This directory contains Git hooks for code quality and commit standards.

## Automatic Installation

Hooks are **automatically configured** when you enter the workspace directory (via mise `enter` hook). Just run:

```bash
cd /path/to/metorial-workspace
# Hooks are now active!
```

## Manual Installation

If needed, you can manually install/uninstall hooks:

```bash
# Install hooks
mise run git:hooks:install

# Uninstall hooks (revert to default)
mise run git:hooks:uninstall
```

Or directly via git:

```bash
git config --local core.hooksPath .githooks
```

## Available Hooks

### pre-commit

Runs `drift check --staged` on staged TypeScript/JavaScript files before each commit.

- **Fails on:** Errors only (warnings are allowed)
- **Bypass:** `git commit --no-verify` (not recommended)

### pre-push

Runs quality gate checks before pushing to remote.

- **Protected branches (main/master/develop):** Full gate, fails on warnings
- **Feature branches:** Lighter checks, fails on errors only
- **Bypass:** `git push --no-verify` (not recommended)

### commit-msg

Validates commit messages follow [Conventional Commits](https://www.conventionalcommits.org/) format.

- **Format:** `type(scope): description`
- **Valid types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`
- **Breaking changes:** Add `!` after type/scope (e.g., `feat(api)!: breaking change`)
- **Bypass:** `git commit --no-verify` (not recommended)

## Why Not Husky?

This workspace uses [mise](https://mise.jdx.dev) as the primary task orchestrator rather than npm at the root level. The mise `enter` hook provides automatic git hook configuration without additional dependencies.

For reference, see:
- [Git Hooks without Husky](https://dev.to/azu/git-hooks-without-extra-dependencies-like-husky-in-node-js-project-jjp)
- [Native Git core.hooksPath](https://git-scm.com/docs/git-config#Documentation/git-config.txt-corehooksPath)

## Requirements

- **mise** must be installed and active
- **Drift** must be installed (`mise install`)

## Troubleshooting

If hooks fail unexpectedly:

1. Run `drift check --staged --verbose` to see detailed output
2. Run `drift gate --verbose` to see gate details
3. Check that Drift is installed: `drift --version`
4. Verify hooks are active: `git config core.hooksPath`

If Drift is not installed, the pre-commit and pre-push hooks will skip silently with a warning.
