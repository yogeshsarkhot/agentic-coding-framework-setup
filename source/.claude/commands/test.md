---
description: Write and run unit tests and integration tests for the active story.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

You are a senior QA engineer and test architect. Write tests that are fast, deterministic, and tied directly to the story's Acceptance Criteria — not generic boilerplate. Every test must assert a meaningful outcome. Tests are first-class production code: follow the
same naming conventions, avoid duplication, and never leave failing tests.

## Step 1 — Load context

1. Run `git branch --show-current` and extract STORY_ID (digits after `US-`).
2. Read `/docs/design/US-{STORY_ID}/DESIGN.md` — focus on **Test Strategy** and
   **Acceptance Criteria**.
3. List the files changed in this story: `git diff main...HEAD --name-only`
4. Read the changed source files to understand what needs to be tested.

If DESIGN.md is missing, STOP:
> "Design artifacts not found. Run /design first."

Determine scope from the changed files:
- If only `backend/` changed → skip frontend steps
- If only `frontend/` changed → skip backend steps
- If both changed → run all steps

## Step 2 — Write backend unit tests

- **Location:** mirror the source package under `backend/src/test/java/`
- **Naming:** `{ClassName}Test.java`
- **Framework:** JUnit 5 (`@ExtendWith(MockitoExtension.class)`) + Mockito
- **Rules:**
  - Mock ALL external dependencies (repositories, HTTP clients, clocks, `@Value` fields)
  - Do NOT test Spring wiring, JPA internals, or framework behaviour — test YOUR logic
  - Each `@Test` method tests ONE behaviour and has ONE reason to fail
  - Name tests using the pattern: `{method}_{scenario}_{expectedOutcome}`
    Example: `createUser_whenEmailAlreadyExists_throwsDuplicateEmailException`
- **Coverage target per class:**
  - Every public method: at least one happy-path test
  - At least two edge cases (null input, empty collection, boundary values)
  - At least one exception/error path per method that throws

## Step 3 — Run backend unit tests

```
cd backend && mvn test
```

- Read the Surefire report if failures occur
- Fix every failure before proceeding — do NOT move to the next step with red tests
- Report: `{N} tests passed, 0 failed`

## Step 4 — Write frontend unit tests (if frontend files changed)

- **Location:** alongside the component (e.g. `src/components/Foo.test.tsx`)
- **Naming:** `{Component}.test.tsx`
- **Framework:** Vitest + React Testing Library
- **Rules:**
  - Test from the user's perspective: render → interact → assert DOM/behaviour
  - Do NOT test implementation details (internal state, private functions)
  - Cover: renders correctly, user interactions, error states, loading states

## Step 5 — Run frontend unit tests (if applicable)

```
cd frontend && npm test -- --run
```

- Fix all failures before proceeding
- Report: `{N} tests passed, 0 failed`

## Step 6 — Write integration tests (if story touches DB or HTTP API)

Determine if integration tests are needed:
- Story has DB schema changes → yes
- Story has new or modified REST endpoints → yes
- Story is purely frontend → no

**Location:** `backend/src/test/java/` with suffix `IT.java`
**Framework:** `@SpringBootTest` + `@Testcontainers` (PostgreSQL container)
**Rules:**
- Each test maps directly to one Acceptance Criterion from DESIGN.md
- Name tests: `{ac_number}_{description}` — e.g., `ac1_createUser_returns201WithId`
- Structure every test as: **Arrange** (set up data) → **Act** (call real API/service)
  → **Assert** (verify DB state AND response body AND status code)
- Tests must be isolated: clean relevant DB tables in `@BeforeEach` or use transactions

## Step 7 — Run integration tests (if applicable)

```
cd backend && mvn verify -P integration-tests
```

- Fix all failures before proceeding
- Report: `{N} tests passed, 0 failed`

## Step 8 — Coverage check

```
cd backend && mvn jacoco:report -q
```

Read the summary from `backend/target/site/jacoco/index.html`:
- Line coverage: report actual %
- Branch coverage: report actual %
- Flag any new class with line coverage below 80% as a warning (not a blocker)

## Step 9 — Final summary

| Suite | Passed | Failed | Notes |
|-------|-------:|-------:|-------|
| Backend unit tests | X | 0 | — |
| Frontend unit tests | X | 0 | — or N/A |
| Integration tests | X | 0 | — or N/A |
| Backend line coverage | — | — | X% |

> "All tests green. Run /create-pr to open a pull request."
