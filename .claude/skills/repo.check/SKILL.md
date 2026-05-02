---
name: repo.check
description: Check a GitHub repository for compliance with AI agent scaffolding best practices (Tiers 1-4). Accepts a GitHub URL or local path and generates a pass/warn/fail report.
allowed-tools: Bash, Read
user-invocable: true
argument-hint: "<github-url-or-local-path> — e.g., 'https://github.com/opendatahub-io/odh-dashboard' or '/Users/me/workspace/my-repo'"
---

# Repo Scaffolding Compliance Checker

Check a repository against the best practices from the [WG Agentic SDLC repo scaffolding guide](https://gitlab.cee.redhat.com/global-engineering/wg-agentic-sdlc/-/blob/main/best-practices/repo-scaffolding-for-ai-agents.md). Covers all four tiers, checking automatable practices at each level.

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

---

## Tier 1 Checks — Do This Week (Highest Impact)

### Check 1.1: Tests Runnable with a Single Command

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

### Check 1.2: Minimal Context File

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

### Check 1.3: Lint and Type Checking on Single Files

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

### Check 1.4: CI Quality Gates

**What to look for:**

1. **CI configuration exists**:
   - `.github/workflows/` directory with `*.yml` or `*.yaml` files
   - `.gitlab-ci.yml`
   - `Jenkinsfile`
   - `.tekton/`

2. **CI includes lint and test steps**: if CI workflow files are found, read them and check for references to lint commands (`eslint`, `ruff`, `golangci-lint`, `flake8`, `pylint`) and test commands (`pytest`, `npm test`, `go test`, `cargo test`, `make test`)

**Scoring:**
- **PASS**: CI config exists AND includes both lint and test steps
- **WARN**: CI config exists but missing either lint or test steps
- **FAIL**: no CI configuration found

---

## Tier 2 Checks — Do This Month (Significant Impact)

### Check 2.1: Pattern References for Common Changes

**What to look for:**

1. **Skills directory**: `.claude/skills/` directory containing one or more subdirectories with `SKILL.md` files
2. **Pattern section in context file**: check CLAUDE.md/AGENTS.md for a section referencing examples or patterns (grep for `pattern`, `example`, `follow the pattern`, `reference`, `template`)

**Scoring:**
- **PASS**: `.claude/skills/` directory exists with at least one SKILL.md
- **WARN**: context file mentions patterns/examples but no dedicated skills directory
- **FAIL**: no pattern references or skills found

### Check 2.2: Hooks for Deterministic Enforcement

**What to look for:**

1. **Claude Code / Codex hooks**: `.claude/settings.json` containing a `hooks` key with `PreToolUse` or `PostToolUse` entries
2. **Pre-commit hooks**: `.pre-commit-config.yaml` exists with at least one hook configured

**Scoring:**
- **PASS**: Claude Code hooks configured (`.claude/settings.json` with hooks) OR `.pre-commit-config.yaml` exists
- **WARN**: only `.pre-commit-config.yaml` exists (no agent-specific hooks)
- **FAIL**: no hook configuration found

### Check 2.3: Design Intent Documentation

**What to look for:**

1. **Design documentation directory**: `docs/design/`, `docs/architecture/`, or `docs/adr/` directory with markdown files
2. **Architecture file**: `ARCHITECTURE.md` at repo root
3. **ADR directory**: `docs/adr/` or `adr/` directory with decision records

**Scoring:**
- **PASS**: design docs directory exists with files AND (`ARCHITECTURE.md` exists OR ADR directory exists)
- **WARN**: only one of: design docs directory, ARCHITECTURE.md, or ADR directory
- **FAIL**: none found

### Check 2.4: Type System Enforcement

**What to look for (language-specific strict mode):**

1. **JS/TS**: `tsconfig.json` with `"strict": true` or `"noImplicitAny": true`
2. **Python**: `pyproject.toml` with `[tool.mypy]` section containing `strict = true` or `disallow_untyped_defs = true`, or `mypy.ini` / `pyrightconfig.json` exists
3. **Go**: inherently typed — auto-PASS
4. **Rust**: inherently typed — auto-PASS

**Scoring:**
- **PASS**: strict type checking enabled (or inherently typed language)
- **WARN**: type checker config exists but strict mode not enabled
- **FAIL**: no type checker configuration found

### Check 2.5: Predictable File Organization

**What to look for:**

1. **Source directory**: `src/` or language-standard source directory (`cmd/` + `internal/` for Go, `lib/` for Ruby)
2. **Test directory mirrors source**: `tests/` or `test/` or `__tests__/` at same level as source, or `*_test.go` files alongside source (Go convention)
3. **Consistent naming**: check for mixed patterns (e.g., both `camelCase.ts` and `snake_case.ts` in same directory)

**Scoring:**
- **PASS**: standard source layout AND test directory exists mirroring source
- **WARN**: source or test directory exists but organization is non-standard
- **FAIL**: no recognizable project layout

---

## Tier 3 Checks — Do This Quarter (Large/Complex Repos)

### Check 3.1: Component-Level Context Files

**What to look for:**

1. **Path-scoped rules**: `.claude/rules/` directory with markdown files containing `paths:` frontmatter
2. **Nested context files**: `CLAUDE.md` files in subdirectories (not just root)

**Scoring:**
- **PASS**: `.claude/rules/` directory exists with at least one file
- **WARN**: nested CLAUDE.md files exist in subdirectories (less structured than rules)
- **FAIL**: no component-level context — only root-level context file
- **N/A**: repo is small (<100 files) — component-level context not needed

### Check 3.2: Machine-Readable API Surfaces

**What to look for:**

1. **OpenAPI specs**: `openapi.yaml`, `openapi.json`, `swagger.yaml`, `swagger.json`, or files matching `**/openapi.*`, `**/swagger.*`
2. **Protobuf definitions**: `*.proto` files
3. **Kubernetes CRDs**: files containing `kind: CustomResourceDefinition` or in a `crds/` directory
4. **GraphQL schemas**: `*.graphql`, `schema.graphql`

**Scoring:**
- **PASS**: at least one machine-readable API spec found
- **WARN**: API-like code exists (routes, handlers, endpoints) but no spec files
- **FAIL**: no API specs found
- **N/A**: repo does not appear to expose APIs (no route/handler/endpoint patterns)

### Check 3.3: Architectural Boundary Lint Rules

**What to look for:**

1. **Import restrictions in linter config**:
   - JS/TS: `.eslintrc*` or `eslint.config.*` with `no-restricted-imports` rules
   - Python: `pyproject.toml` with `[tool.ruff.lint.per-file-ignores]` or `[tool.pylint]` import rules, or `.importlinter` config
   - Go: `.golangci.yml` with `depguard` or `gomodguard` linter enabled
2. **Dependency enforcement**: `dependency-cruiser` config (`.dependency-cruiser.cjs`), or `import-boundary` type rules

**Scoring:**
- **PASS**: import restriction or boundary rules configured in linter
- **WARN**: linter exists but no boundary/import restriction rules
- **FAIL**: no boundary enforcement found
- **N/A**: repo is small enough that boundaries are not needed (<20 files)

---

## Tier 4 Checks — Advanced (At Scale)

### Check 4.1: On-Demand Skills System

**What to look for:**

1. **Skills directory**: `.claude/skills/` with **3 or more** subdirectories, each containing a `SKILL.md`
2. **Context file size**: if root context file is >150 lines AND no skills directory exists, the context should be split

**Scoring:**
- **PASS**: `.claude/skills/` exists with 3+ skills (context is properly split)
- **WARN**: 1-2 skills exist, or context file is >150 lines without skills
- **FAIL**: no skills system and context file is >150 lines
- **N/A**: context file is ≤150 lines (splitting not needed)

---

## Generate Report

Produce the following report. Use exact formatting.

```markdown
## Repo Scaffolding Compliance Report

**Repository:** {owner}/{repo}
**Primary Language:** {language}
**Date:** {YYYY-MM-DD}

### Tier 1 — Do This Week (Highest Impact)

| # | Practice | Status | Details |
|---|---|---|---|
| 1.1 | Single-command tests | {PASS/WARN/FAIL} | {brief explanation} |
| 1.2 | Minimal context file | {PASS/WARN/FAIL} | {brief explanation} |
| 1.3 | Single-file lint/type-check | {PASS/WARN/FAIL} | {brief explanation} |
| 1.4 | CI quality gates | {PASS/WARN/FAIL} | {brief explanation} |

### Tier 2 — Do This Month

| # | Practice | Status | Details |
|---|---|---|---|
| 2.1 | Pattern references / skills | {PASS/WARN/FAIL} | {brief explanation} |
| 2.2 | Hooks for enforcement | {PASS/WARN/FAIL} | {brief explanation} |
| 2.3 | Design intent docs | {PASS/WARN/FAIL} | {brief explanation} |
| 2.4 | Type system enforcement | {PASS/WARN/FAIL} | {brief explanation} |
| 2.5 | Predictable file organization | {PASS/WARN/FAIL} | {brief explanation} |

### Tier 3 — Do This Quarter

| # | Practice | Status | Details |
|---|---|---|---|
| 3.1 | Component-level context | {PASS/WARN/FAIL/N/A} | {brief explanation} |
| 3.2 | Machine-readable API specs | {PASS/WARN/FAIL/N/A} | {brief explanation} |
| 3.3 | Boundary lint rules | {PASS/WARN/FAIL/N/A} | {brief explanation} |

### Tier 4 — Advanced

| # | Practice | Status | Details |
|---|---|---|---|
| 4.1 | On-demand skills system | {PASS/WARN/FAIL/N/A} | {brief explanation} |

**Summary: {T1_pass}/4 Tier 1 | {T2_pass}/5 Tier 2 | {T3_pass}/{T3_applicable} Tier 3 | {T4_pass}/{T4_applicable} Tier 4**
```

If there are any WARN or FAIL results, add a Recommendations section grouped by tier:

```markdown
### Recommendations

**Tier 1 (address this week):**
- **{Practice}**: {specific, actionable recommendation}

**Tier 2 (address this month):**
- **{Practice}**: {specific, actionable recommendation}

**Tier 3 (address this quarter):**
- **{Practice}**: {specific, actionable recommendation}
```

Keep recommendations concrete and actionable. Examples:
- "Add a `CLAUDE.md` with build and test commands. Keep it under 150 lines."
- "Document single-file lint commands in CLAUDE.md: `ruff check <file>`"
- "Add `.claude/skills/` directory with SKILL.md files for common change patterns (e.g., add-api-endpoint, add-migration)."
- "Create `ARCHITECTURE.md` documenting module boundaries and key design decisions."
- "Enable `strict: true` in `tsconfig.json` for full type enforcement."
- "Add `.claude/settings.json` with PostToolUse hooks for auto-formatting on edit."

## Notes

- Do NOT clone the repository. Use `gh api` for remote repos and filesystem commands for local repos.
- Run all checks sequentially. Do not parallelize.
- Keep the output concise. The report should fit on one screen.
- If `gh` CLI is not authenticated or the repo is not accessible, report the error and stop.
- For Tier 3 checks, mark as N/A when the repo is too small for the practice to be relevant.
- Practices that cannot be automated are excluded: 3.4 (continuous improvement loop) and 4.2 (multi-agent workflows).
