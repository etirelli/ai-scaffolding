---
name: repo.check
description: Check a GitHub repository for compliance with AI agent scaffolding best practices (Tier 1). Accepts a GitHub URL or local path and generates a pass/warn/fail report.
allowed-tools: Bash, Read
user-invocable: true
argument-hint: "<github-url-or-local-path> — e.g., 'https://github.com/opendatahub-io/odh-dashboard' or '/Users/me/workspace/my-repo'"
---

# Repo Scaffolding Tier 1 Compliance Checker

Check a repository against the four Tier 1 best practices from the [WG Agentic SDLC repo scaffolding guide](https://gitlab.cee.redhat.com/global-engineering/wg-agentic-sdlc/-/blob/main/best-practices/repo-scaffolding-for-ai-agents.md). These are the highest-impact, lowest-effort practices that teams should complete within a week.

## Input

The user provides either:
- A **GitHub URL** (e.g., `https://github.com/opendatahub-io/odh-dashboard`)
- A **local filesystem path** (e.g., `/Users/me/workspace/my-repo`)

## Execution

### Step 0: Determine Access Method

**If the input is a GitHub URL:**
- Extract `{owner}` and `{repo}` from the URL
- Use `gh` CLI for all checks: `gh api repos/{owner}/{repo}/...`
- Fetch the full file tree once and reuse it:
  ```bash
  gh api "repos/{owner}/{repo}/git/trees/HEAD?recursive=1" --jq '.tree[].path'
  ```
- To read file contents:
  ```bash
  gh api "repos/{owner}/{repo}/contents/{path}" --jq '.content' | base64 -d
  ```

**If the input is a local path:**
- Verify the path exists and is a git repository
- Use `ls`, `find`, `cat`, `wc`, `grep` directly

Store the access method and repo identifier for use in all subsequent checks.

### Step 1: Detect Primary Language

Determine the repo's primary language by examining file extensions in the tree:

| Files Found | Language |
|---|---|
| `*.ts`, `*.tsx`, `*.js`, `*.jsx`, `package.json` | JavaScript/TypeScript |
| `*.py`, `pyproject.toml`, `setup.py`, `setup.cfg` | Python |
| `*.go`, `go.mod` | Go |
| `*.rs`, `Cargo.toml` | Rust |
| `*.java`, `pom.xml`, `build.gradle` | Java |

This informs which specific config files to look for in each check.

### Step 2: Check 1.1 — Tests Runnable with a Single Command

**What to look for:**

1. **Test configuration** (language-specific):
   - JS/TS: `package.json` with `scripts.test`, `jest.config.*`, `vitest.config.*`, `.mocharc.*`
   - Python: `pyproject.toml` with `[tool.pytest]`, `pytest.ini`, `tox.ini`, `setup.cfg` with `[tool:pytest]`
   - Go: `go.mod` (Go has built-in `go test`)
   - Rust: `Cargo.toml` (Rust has built-in `cargo test`)
   - General: `Makefile` with `test` target

2. **Test files exist**:
   - JS/TS: `*.test.ts`, `*.spec.ts`, `__tests__/` directory
   - Python: `tests/` or `test/` directory, `test_*.py` files
   - Go: `*_test.go` files
   - Rust: `#[test]` in source or `tests/` directory

3. **Test command documented**: check CLAUDE.md, AGENTS.md, README.md for test command references (look for patterns like `npm test`, `pytest`, `go test`, `make test`, `cargo test`)

**Scoring:**
- **PASS**: test config exists AND test files exist
- **WARN**: test files exist but no single-command runner or no documented test command
- **FAIL**: no test files or test infrastructure found

### Step 3: Check 1.2 — Minimal Context File

**What to look for:**

1. **Context file exists** at repo root:
   - `CLAUDE.md` (Claude Code)
   - `AGENTS.md` (cross-tool)
   - `.cursor/rules` (Cursor)
   - `.github/copilot-instructions.md` (GitHub Copilot)

2. **If found, assess quality:**
   - Count lines (target: under 150, acceptable: 150-300, too long: >300)
   - Check for build/test commands section (grep for patterns: `build`, `test`, `lint`, `npm`, `pytest`, `make`, `go test`, `cargo`)
   - Check for key conventions section

**Scoring:**
- **PASS**: context file exists, under 300 lines, and contains build/test commands
- **WARN**: context file exists but either >300 lines OR missing build/test commands
- **FAIL**: no context file found

### Step 4: Check 1.3 — Lint and Type Checking on Single Files

**What to look for:**

1. **Linter configuration** (language-specific):
   - JS/TS: `.eslintrc*`, `eslint.config.*`, `biome.json`
   - Python: `ruff.toml`, `pyproject.toml` with `[tool.ruff]` or `[tool.flake8]`, `.flake8`, `.pylintrc`
   - Go: `.golangci.yml`, `.golangci.yaml`
   - Rust: `rustfmt.toml`, `clippy.toml`

2. **Type checker configuration**:
   - JS/TS: `tsconfig.json`
   - Python: `mypy.ini`, `pyproject.toml` with `[tool.mypy]`, `pyrightconfig.json`
   - Go: built-in (go vet)
   - Rust: built-in

3. **Single-file commands documented**: check CLAUDE.md/AGENTS.md for single-file lint commands (e.g., `npx eslint --fix path/to/file`, `ruff check path/to/file`, `golangci-lint run path/to/file`)

**Scoring:**
- **PASS**: linter configured AND single-file lint commands documented in context file
- **WARN**: linter configured but no single-file commands documented
- **FAIL**: no linter configuration found

### Step 5: Check 1.4 — CI Quality Gates

**What to look for:**

1. **CI configuration exists**:
   - `.github/workflows/` directory with `*.yml` or `*.yaml` files
   - `.gitlab-ci.yml`
   - `Jenkinsfile`
   - `.tekton/`

2. **CI includes lint and test steps**: if GitHub Actions workflows are found, read the workflow files and check for references to lint commands (`eslint`, `ruff`, `golangci-lint`, `flake8`, `pylint`) and test commands (`pytest`, `npm test`, `go test`, `cargo test`, `make test`)

**Scoring:**
- **PASS**: CI config exists AND includes both lint and test steps
- **WARN**: CI config exists but missing either lint or test steps
- **FAIL**: no CI configuration found

### Step 6: Generate Report

Produce the following report. Use exact formatting.

```markdown
## Repo Scaffolding Tier 1 Compliance Report

**Repository:** {owner}/{repo}
**Primary Language:** {language}
**Date:** {YYYY-MM-DD}

| # | Practice | Status | Details |
|---|---|---|---|
| 1.1 | Single-command tests | {PASS/WARN/FAIL} | {brief explanation} |
| 1.2 | Minimal context file | {PASS/WARN/FAIL} | {brief explanation} |
| 1.3 | Single-file lint/type-check | {PASS/WARN/FAIL} | {brief explanation} |
| 1.4 | CI quality gates | {PASS/WARN/FAIL} | {brief explanation} |

**Overall: {N}/4 passing**
```

If there are any WARN or FAIL results, add a Recommendations section:

```markdown
### Recommendations

- **{Practice}**: {specific, actionable recommendation}
```

Keep recommendations concrete and actionable. For example:
- "Add a `CLAUDE.md` with build and test commands. Keep it under 150 lines."
- "Document single-file lint commands in CLAUDE.md: `npx eslint --fix <file>`"
- "Add a `test` job to the GitHub Actions workflow in `.github/workflows/ci.yml`"

## Notes

- Do NOT clone the repository. Use `gh api` for remote repos and filesystem commands for local repos.
- Run all checks sequentially. Do not parallelize.
- Keep the output concise. The report should fit on one screen.
- If `gh` CLI is not authenticated or the repo is not accessible, report the error and stop.
