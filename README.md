# repo-scaf

Claude Code plugin that checks GitHub repositories for compliance with [AI agent scaffolding best practices](https://gitlab.cee.redhat.com/global-engineering/wg-agentic-sdlc/-/blob/main/best-practices/repo-scaffolding-for-ai-agents.md) (Tier 1).

## Usage

```
/repo.check <github-url-or-local-path>
```

## What it checks

| Practice | What it looks for |
|---|---|
| 1.1 Single-command tests | Test config files, test directories, documented test commands |
| 1.2 Minimal context file | CLAUDE.md / AGENTS.md presence, line count, build/test commands |
| 1.3 Single-file lint/type-check | Linter config, type checker config, documented single-file commands |
| 1.4 CI quality gates | CI config files, lint/test steps in workflows |

## Installation

```bash
claude plugin add /path/to/repo-scaf
```

## License

Apache-2.0
