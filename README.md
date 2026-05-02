# AI Scaffolding Skills

Claude Code plugin that checks GitHub repositories for compliance with [AI agent scaffolding best practices](https://gitlab.cee.redhat.com/global-engineering/wg-agentic-sdlc/-/blob/main/best-practices/repo-scaffolding-for-ai-agents.md) across all four tiers.

## Installation

```
/plugin marketplace add etirelli/ai-scaffolding
/plugin install ai-scaffolding
```

## Usage

```
/repo.check <github-url-or-local-path>
```

Examples:

```
/repo.check https://github.com/opendatahub-io/odh-dashboard
/repo.check /Users/me/workspace/my-repo
```

## What it checks

### Tier 1 — Do This Week (Highest Impact)

| # | Practice | What it looks for |
|---|---|---|
| 1.1 | Single-command tests | Test config files, test directories, documented test commands |
| 1.2 | Minimal context file | CLAUDE.md / AGENTS.md presence, line count, build/test commands |
| 1.3 | Single-file lint/type-check | Linter config, type checker config, documented single-file commands |
| 1.4 | CI quality gates | CI config files, lint/test steps in workflows |

### Tier 2 — Do This Month

| # | Practice | What it looks for |
|---|---|---|
| 2.1 | Pattern references | `.claude/skills/` directory or pattern section in context file |
| 2.2 | Hooks for enforcement | `.claude/settings.json` hooks or `.pre-commit-config.yaml` |
| 2.3 | Design intent docs | `docs/design/`, `ARCHITECTURE.md`, ADR directory |
| 2.4 | Type system enforcement | Strict mode enabled in type checker config |
| 2.5 | Predictable file org | Standard source layout, test directory mirroring |

### Tier 3 — Do This Quarter

| # | Practice | What it looks for |
|---|---|---|
| 3.1 | Component-level context | `.claude/rules/` with path-scoped files |
| 3.2 | Machine-readable APIs | OpenAPI specs, protobuf files, CRD YAMLs, GraphQL schemas |
| 3.3 | Boundary lint rules | Import restrictions in linter config |

### Tier 4 — Advanced

| # | Practice | What it looks for |
|---|---|---|
| 4.1 | On-demand skills | `.claude/skills/` with 3+ skills for context splitting |

Each practice is scored **PASS**, **WARN**, **FAIL**, or **N/A** with a brief explanation and actionable recommendations.

## License

Apache-2.0
