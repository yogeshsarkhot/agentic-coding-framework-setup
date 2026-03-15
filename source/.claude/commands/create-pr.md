---
description: Stage, commit, push, and create a Pull Request in Azure DevOps.
allowed-tools: Bash(git *), mcp__azure-devops__pull_requests_create,
               mcp__azure-devops__work_items_update, Read, Glob
---

You are a senior developer preparing a production-bound pull request. The PR must be clean, traceable, and ready for review. Never take shortcuts that hide problems.

## Step 1 — Load context

1. Run `git branch --show-current` and extract STORY_ID (digits after `US-`).
2. Read `/docs/design/US-{STORY_ID}/DESIGN.md` to retrieve the story title, solution summary, and Acceptance Criteria list.

## Step 2 — Safety checks before staging

Run `git status` and inspect every changed and untracked file.

**Sensitive file scan — STOP if any of these are present:**
- `.env`, `*.env`, `*.env.*`
- Files matching: `*secret*`, `*password*`, `*credential*`, `*token*`
- Files matching: `*.key`, `*.pem`, `*.p12`, `*.jks`
- Any file not in `.gitignore` that contains raw credentials

If a sensitive file is detected, STOP:
> "⛔ Sensitive file detected: {filename}. Add it to .gitignore and verify it was never committed to history before continuing."

Run `git diff --stat HEAD` and display the scope of changes so the user can see what will be committed.

## Step 3 — Stage and commit

Stage only relevant directories — do NOT use `git add -A`:
```
git add backend/ frontend/ docs/ docker/ terraform/ .github/ k8s/
```

Exclude from staging (do not add):
- `.env*`, IDE config (`.idea/`, `.vscode/` unless the team tracks these), build artifacts (`target/`, `dist/`, `node_modules/`)

**Generate the commit message** using this format:
```
{type}({scope}): {story title in lower-case imperative mood}

- {key change 1}
- {key change 2}
- {key change 3}

Closes AB#{STORY_ID}
```

Where:
- `type` is one of: `feat` | `fix` | `chore` | `docs` | `test` | `refactor`
- `scope` is the primary layer changed: `backend` | `frontend` | `infra` | `db` | `docs`
- Imperative mood: "add user login" not "added user login"

Show the full commit message to the user and ask:
> "Commit with this message? (yes / edit / abort)"

- If `edit`: ask the user to provide the corrected message, then use it
- If `abort`: STOP

Run: `git commit -m "{approved message}"`

## Step 4 — Push

```
git push --set-upstream origin HEAD
```

If push is rejected due to remote divergence:
```
git pull --rebase origin main
git push
```
If the rebase fails (conflicts): STOP and report:
> "Push failed due to conflicts with main. Resolve conflicts manually, then re-run /create-pr."

## Step 5 — Create Pull Request via Azure DevOps MCP

Use `mcp__azure-devops__pull_requests_create` with:

- **Title:** `feat: US-{STORY_ID} — {story title}`
- **Source branch:** current branch
- **Target branch:** `main`
- **Reviewers:** leave empty (team will self-assign)
- **Work item link:** US-{STORY_ID}
- **Description** (structured markdown):

```markdown
## Summary
{2–3 sentences from DESIGN.md solution approach}

## Acceptance Criteria
{List each criterion with a ✅ to confirm it is implemented}

## Changes
{List of key files created or modified, grouped by layer}

## Test Evidence
- Backend unit tests: {N} passed, 0 failed
- Integration tests: {N} passed, 0 failed (or N/A)
- Frontend tests: {N} passed, 0 failed (or N/A)
```

## Step 6 — Update work item

Set State = `Resolved` on work item US-{STORY_ID} via MCP.

## Step 7 — Confirm

Print the PR URL, then:
> "PR created and US-{STORY_ID} set to Resolved. Run /review-pr to self-review,
> or share the URL for team review."
