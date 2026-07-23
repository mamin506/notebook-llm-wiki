---
name: create-pr
description: "Use when: you need to prepare local changes for review, split them into atomic commits, and open a PR with sign-offs using git and gh."
---

# Create PR

## When to Use
- You need to turn local changes into a reviewable PR.
- The work should be split into one or more atomic commits.
- You need a compact, terminal-first workflow with sign-offs.

## Procedure
1. Check repo state.
   - Run `git branch --show-current`.
   - If on `main`, create a new feature branch before committing. Choose a short descriptive name.
2. Inspect changes.
   - Run `git status --short` and `git diff --stat`.
   - If no changes, stop.
3. Create commits.
   - Group changes into one or more atomic commits.
   - Each commit must be self-contained, easy to review, and safe to revert.
   - Use clear commit messages.
   - Add `Signed-off-by: Claude Sonnet 4.5 <noreply@anthropic.com>` to every commit message.
4. Verify history.
   - Run `git log --oneline -n <N>` and confirm the commits are sensible.
   - If needed, amend or split commits.
5. Create PR.
   - Push the branch if needed.
   - Use `gh pr create --title "..." --body "..."`.
   - End the PR body with `Signed-off-by: Claude Sonnet 4.5 <noreply@anthropic.com>`.
6. Hand off.
   - Leave the PR for human review and summarize the branch, commits, and any risks.

## Quality Checks
- Not left on `main`.
- Each commit is atomic and revertible.
- Every commit and the PR body include a sign-off.
- PR title/body are concise and clear.
