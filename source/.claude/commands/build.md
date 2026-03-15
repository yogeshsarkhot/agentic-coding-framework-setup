---
description: Implement the active user story following the approved design.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

You are a senior full-stack developer. Implement the story precisely as designed — no gold-plating, no scope creep, no refactoring of unrelated code. Read before you write. Compile after every file change to catch errors immediately.

## Step 1 — Load context

1. Run `git branch --show-current` and extract STORY_ID (digits after `US-`).
2. Read `CLAUDE.md` for conventions, build commands, and ARB design rules.
3. Read `/docs/design/US-{STORY_ID}/TASKS.md` for the ordered task list.
4. Read `/docs/design/US-{STORY_ID}/DESIGN.md` for solution approach, API contracts, and database changes.

If either design file is missing, STOP:
> "Design artifacts not found at /docs/design/US-{STORY_ID}/. Run /design first."

If TASKS.md contains no unchecked tasks (`- [ ]`), STOP:
> "All tasks in TASKS.md are already checked. If this is unexpected, review the file manually."

## Step 2 — Implement each task in order

For each unchecked task (`- [ ]`) in TASKS.md, in sequence:

**a) Read first**
Read all existing source files relevant to this task before writing anything. Understand the existing patterns (naming, package structure, error handling style) and follow them.

**b) Implement**
Apply the change. Strictly enforce all of the following — no exceptions:

| Rule | Detail |
|------|--------|
| No hardcoded secrets | Use `${ENV_VAR}` in application.yml or environment injection only |
| No `System.out.println()` | Use SLF4J logger: `log.info("msg", kv("field", value))` |
| No direct DDL | All schema changes via a new Flyway migration SQL file |
| ABAC on endpoints | Every new `@RestController` method needs `@PreAuthorize(...)` |
| Structured logging fields | Include `traceId`, `spanId`, `correlationId` on every log statement |
| DTOs separate from entities | Never return a JPA `@Entity` directly from a controller |

**c) Compile immediately after each file change**
- Java file changed: `cd backend && mvn compile -q`
  - If compile fails: **fix it now** before touching the next task. Do not accumulate errors.
- TypeScript file changed: `cd frontend && npm run build --quiet`
  - If build fails: **fix it now** before continuing.

**d) Mark the task complete**
Edit TASKS.md: change `- [ ]` → `- [x]` for the completed task.

## Step 3 — Full build verification

After all tasks are complete, run the full build:

```
cd backend && mvn clean verify -q
```
(This compiles, runs unit tests, and checks Checkstyle/PMD if configured.)

```
cd frontend && npm ci && npm run build
```

If either fails:
- Read the full error output
- Fix the root cause
- Re-run the failing build before reporting completion
- Do NOT report success until both builds are green

## Step 4 — Report

Print a completion summary:

```
## Build Complete — US-{STORY_ID}

### Files created
- path/to/NewFile.java
- ...

### Files modified
- path/to/ExistingFile.java
- ...

### Build results
- Backend:  ✅ mvn clean verify — {N} tests passed
- Frontend: ✅ npm run build — no errors

### Notes
{Any warnings, deferred decisions, or follow-up items for the test phase}
```

> "Implementation complete. Run /test to write and run tests."
