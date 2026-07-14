---
description: Create a git commit with user-approved commit message
model: openai/gpt-5.6-terra-fast
---

Create a git commit.

## Staged changes (already provided — do NOT re-run git diff)

!`git diff --staged`

## Instructions

1. If nothing staged → Exit with error: "Nothing staged. Stage files first."
2. Use conventional-commit skill to generate a commit message. Output ONLY the message — no markdown, no explanation, no extra text.
3. Run `git commit -m "{message}"` immediately after generating the message.
4. If commit fails, show the error and stop.

## Rules

- Pre-commit hook failure: show the error and stop
