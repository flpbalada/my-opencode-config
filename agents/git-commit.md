---
description: Generate conventional commit messages using Claude AI. Use when the user says "commit", mycommit", "commit my changes", wants to create a git commit, or needs an automated commit message based on staged changes.
mode: subagent
model: anthropic/claude-sonnet-4-5-20250929
tools:
  bash: true
  read: true
  grep: true
  list: true
  lsp: true
---

# MyCommit - Automated Conventional Commits

Generate clean, conventional commit messages by analyzing git diffs with Claude AI.

## When to Use

Trigger this skill when the user:
- Says "mycommit" or "/mycommit"
- Asks to "commit my changes" or "create a commit"
- Wants an AI-generated commit message
- Needs to commit with conventional commit format

## Workflow

Execute these steps in order:

### 1. Stage All Changes
```bash
git add .
```

### 2. Run Pre-commit Checks (if package.json exists)
```bash
~/.config/opencode/scripts/run-precommit.sh
```
⚠️ If this fails, stop and report the error to the user.

### 3. Get Git Context
```bash
~/.config/opencode/scripts/get-git-context.sh --staged-only
```
This returns the diff, changed files, and commitlint config (if present).

### 4. Generate Commit Message

Analyze the git context and generate a conventional commit message following these rules:

**Format:** `type(scope): description`

**Types:**
| Type | When to Use |
|------|-------------|
| `feat` | New feature for the user |
| `fix` | Bug fix for the user |
| `docs` | Documentation only changes |
| `style` | Formatting, no code logic change |
| `refactor` | Code restructuring, no new feature or fix |
| `perf` | Performance improvement |
| `test` | Adding/fixing tests |
| `build` | Build system or dependencies |
| `ci` | CI configuration |
| `chore` | Other maintenance |
| `revert` | Reverts a previous commit |

**Rules:**
- Scope is optional but recommended (e.g., `auth`, `api`, `ui`)
- Description: present tense, lowercase, no period at end
- Max 72 characters for the first line
- Focus on the "why" not the "what"

**Examples:**
- `feat(auth): add JWT token validation`
- `fix(api): handle null response in user endpoint`
- `refactor: extract common validation logic`

### 5. Create the Commit
```bash
~/.config/opencode/scripts/create-commit.sh "generated commit message"
```

## Available Scripts

| Script | Purpose |
|--------|---------|
| `get-git-context.sh` | Get diff, files changed, commitlint config |
| `run-precommit.sh` | Run format/lint/typecheck/test |
| `create-commit.sh` | Create commit with given message |

### Script Options

**get-git-context.sh:**
- `--staged-only` - Only show staged changes (default shows all)

**run-precommit.sh:**
- `--skip-tests` - Skip running tests

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Not a git repository / no message |
| 2 | No changes to commit |
| 3 | Pre-commit check failed |
| 5 | Git commit failed |

## User Context

If the user provides context (e.g., "mycommit fixing auth bug"), use it to better understand the intent of the changes when generating the commit message.
