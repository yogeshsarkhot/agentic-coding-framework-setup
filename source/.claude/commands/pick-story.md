---
description: Pull the next unstarted user story from the active sprint and prepare to implement it.
allowed-tools: mcp__azure-devops__work_items_list, mcp__azure-devops__work_items_get, mcp__azure-devops__work_items_update, Bash(git *)
---

You are an expert delivery manager and senior developer. Your goal is to select the highest-priority unstarted story and set it up for immediate implementation with a clean workspace.

## Step 1 — Verify git workspace is clean

Run `git status`. If uncommitted changes or untracked files exist, STOP:
> "Uncommitted changes detected on the current branch. Stash or commit them before
> picking a new story."

## Step 2 — Fetch unstarted stories

Use `work_items_list` to query Work Items of type **"User Story"** in:
- Iteration Path = `SemanticLexiForge\Sprint 1`
- State = `New`
- AssignedTo = `` (unassigned)
- Order by Stack Rank ascending (highest priority first)

If the query returns zero results, print:
> "No unstarted User Stories in Sprint 1. Check the backlog or ask the team to
> populate the sprint."
Then STOP.

## Step 3 — Display the list

Print a table:

| ID | Title | Story Points | Prerequisite |
|----|-------|:------------:|--------------|
| …  | …     | …            | …            |

Then ask:
> "Which story do you want to work on? Enter the ID (or 'cancel' to abort)."

## Step 4 — Load full story details

- If the user types `cancel`, STOP gracefully.
- Fetch the chosen work item with `work_items_get`.
- If the entered ID is not in the displayed list, reply:
  > "Invalid ID. Please enter one of the IDs shown above."
  Re-prompt once. If still invalid, STOP.
- Display the full story: **Title**, **Description**, **Acceptance Criteria**, **Tasks**.
- If the Acceptance Criteria section is empty or vague, flag it:
  > "⚠️ Acceptance Criteria appear incomplete. Clarify with the team before designing."

## Step 5 — Activate the story

Update the work item:
- `State` = `Active`
- `AssignedTo` = current user

## Step 6 — Create and check out the feature branch

**Slugify the title:**
- Lowercase the title
- Replace spaces and special characters with hyphens
- Collapse consecutive hyphens into one
- Truncate to 40 characters maximum
- Example: "Add User Login API (v2)" → `add-user-login-api-v2`

Run:
```
git checkout main && git pull origin main
git checkout -b feat/US-{ID}-{slug}
```

If the branch already exists (exit code non-zero), STOP:
> "Branch feat/US-{ID}-{slug} already exists. Delete it or rename it before picking
> this story again."

## Step 7 — Confirm

> "Story US-{ID} is active and assigned to you.
> Branch feat/US-{ID}-{slug} checked out from latest main.
> Run /design to create the design artifacts."
