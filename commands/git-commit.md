---
description: Generate conventional commit messages using Claude AI. 
agent: build
---

Generate conventional commit messages by analyzing git diffs and creating commits automatically.

## Arguments

$ARGUMENTS — Optional context about the changes (e.g., "fixing auth bug", "refactoring utils")

## Steps

1. **Stage all changes**
   - Run:!`git add .`

2. **Run pre-commit checks**
   - Run:!`~/.config/opencode/scripts/run-precommit.sh`
   - ⚠️ If this fails, stop and report the error to the user

3. **Re-stage after pre-commit fixes**
   - Run:!`git add .`
   - 💡 Pre-commit hooks may auto-fix files (formatting, linting). Re-staging ensures these fixes are included in the commit.

4. **Get git context**
   - Run:!`~/.config/opencode/scripts/get-git-context.sh --staged-only`
   - Returns: diff, changed files, and commitlint config (if present)

5. **Generate commit message**

   Analyze the context and generate a conventional commit following this format:

   **Format:** `type(scope): description`

   **Types:**
   | Type | Use When |
   |------|----------|
   | `feat` | New feature |
   | `fix` | Bug fix |
   | `docs` | Documentation changes |
   | `style` | Formatting, no logic change |
   | `refactor` | Code restructuring |
   | `perf` | Performance improvement |
   | `test` | Adding/fixing tests |
   | `build` | Build system/dependencies |
   | `ci` | CI configuration |
   | `chore` | Other maintenance |
   | `revert` | Reverts previous commit |

   **Rules:**
   - Scope is optional but recommended (e.g., `auth`, `api`, `ui`)
   - Description: present tense, lowercase, no period at end
   - Max 72 characters for first line
   - Focus on the "why" not the "what"

   **Examples:**
   - `feat(auth): add JWT token validation`
   - `fix(api): handle null response in user endpoint`
   - `refactor: extract common validation logic`

   If user provided context in $ARGUMENTS, use it to understand the intent.

6. **Create the commit**
   
   Using the commit message you generated in step 5, run this command with Bash tool:
   
   ```bash
   git commit -m "<commit_message>"
   ```
   
   Replace `<commit_message>` with your generated message. Example:
   ```bash
   git commit -m "feat(auth): add JWT token validation"
   ```

## Scripts

| Script | Purpose |
|--------|---------|
| `get-git-context.sh` | Get diff, files changed, commitlint config |
| `run-precommit.sh` | Run format/lint/typecheck/test |

### Script Options

**get-git-context.sh:**
- `--staged-only` - Only staged changes (default: all)

**run-precommit.sh:**
- `--skip-tests` - Skip running tests
