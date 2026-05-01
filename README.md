# AI Scaffolding Skills

Claude Code plugin that checks GitHub repositories for compliance with [AI agent scaffolding best practices](https://gitlab.cee.redhat.com/global-engineering/wg-agentic-sdlc/-/blob/main/best-practices/repo-scaffolding-for-ai-agents.md) (Tier 1).

## Installation

```bash
claude plugin add etirelli/ai-scaffolding
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

| Practice | What it looks for |
|---|---|
| 1.1 Single-command tests | Test config files, test directories, documented test commands |
| 1.2 Minimal context file | CLAUDE.md / AGENTS.md presence, line count, build/test commands |
| 1.3 Single-file lint/type-check | Linter config, type checker config, documented single-file commands |
| 1.4 CI quality gates | CI config files, lint/test steps in workflows |

Each practice is scored **PASS**, **WARN**, or **FAIL** with a brief explanation and actionable recommendations.

## License

Apache-2.0
