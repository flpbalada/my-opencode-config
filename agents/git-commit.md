---
description: Generate conventional commit messages. Use when the user says "commit", mycommit", "commit my changes", wants to create a git commit, or needs an automated commit message based on staged changes.
tools:
  bash: true
  read: true
  grep: true
  list: true
  question: true
  skill: false
---

## Arguments

$ARGUMENTS — Optional context about the changes (e.g., "fixing auth bug", "refactoring utils")

## Steps

1. **Get git context**
   - Run:!`~/.config/opencode/scripts/run-precommit.sh && git add . && ~/.config/opencode/scripts/get-git-context.sh --staged-only`
   - ⚠️ If this fails, stop and report the error to the user
   - Returns: diff, changed files, and commitlint config (if present)

2. **Generate commit message**

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

3. **Create the commit**
   
   Using the commit message you generated in step 2, run this command with Bash tool:
   
   ```bash
   git commit -m "<commit_message>"
   ```
   
   Replace `<commit_message>` with your generated message. Example:
   ```bash
   git commit -m "feat(auth): add JWT token validation"
   ```
