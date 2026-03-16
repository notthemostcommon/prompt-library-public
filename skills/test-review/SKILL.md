---
name: test-review
description: "Review test suites for acceptance criteria coverage and edge case gaps. Apply when reviewing test files or evaluating test completeness against a story."
---

# Test Review: Coverage Completeness

Systematically verify test completeness against the story and edge case heuristics.

## Step 0: Discover All Test Files

**CRITICAL**: Complete ALL THREE steps below. Missing any step means incomplete review.

Run these searches in order:

1. **Jest tests**: 
   - Run `Glob` with pattern: `**/*.test.ts*`
   - Lists all unit/integration tests in `__tests__/` directory

2. **Playwright tests**:
   - Run `Glob` with pattern: `playwright/**/*.spec.ts`
   - Lists all E2E and Playwright integration tests

3. **Identify relevant tests**:
   - Check which test files were modified/added in the story (git diff, commit history, PR changes)
   - Use `Grep` to find tests containing feature keywords (component names, route paths, error codes)
   - Check which tests import the changed production files
   - Read the story to understand feature scope before selecting tests to review

**STOP**: Do not proceed to Step 1 until you have completed all three steps above.

## Step 1: Map Tests to Acceptance Criteria

**Decompose first.** Story ACs are often compound ("displays error messages" may embed 7 distinct codes). Before mapping, expand each AC into individual testable requirements:

- Split ACs that reference lists, tables, or "mentioned below" into one row per item. Example: "The different error messages are mentioned below" followed by 7 error codes = 7 AC rows, not 1 row saying "all error codes."
- Pull in testable requirements from Notes, Technical Details, and Scope sections
- Use Out of Scope to narrow — exclude items explicitly deferred to other stories or teams
- Treat explicit scenarios described anywhere in the story as ACs

Then for every decomposed AC, determine: which test file:line covers it at each pyramid level (Unit / Integration / E2E)?

- Zero tests for an AC → **CRITICAL GAP**
- Only one pyramid level for error handling or security AC → **PARTIAL**

**Check for spec discrepancies.** Compare test assertion values against story-specified values (messages, codes, labels). Flag mismatches as **DISCREPANCY**. Only list discrepancies, not confirmations. If none found, state "None found." Example: story specifies `"Sign-out failed. Your session has been cleared."` but test asserts `"To finish logging out, please close your browser."` → DISCREPANCY.

## Step 1.5: Verify Happy Path Coverage

For every route, controller, handler, service, or component under test:

- At least one test exercises the successful/default flow?
- For modified files: happy path tests weren't removed?
- For new test files on existing code: happy path included alongside error/edge cases?

No happy path → **CRITICAL GAP**. Error tests without a baseline success test prove the code fails gracefully but not that it works.

| Context | Happy path = |
|---|---|
| API route / controller | Valid request → correct status + body |
| Service / business logic | Valid input → expected output or state change |
| UI component | Default props/state → correct render + interaction |
| CLI / background job | Normal invocation → expected side effects |

**Adjacent code**: If untested code produces inputs consumed by code under test (e.g., routes that emit error codes consumed by a display component), note the gap with a scope tag rather than flagging as CRITICAL for this story.

## Step 2: Verify Error Path Coverage

**Enumerate from implementation**: Read both implementation and test code. Add a row for every `try/catch`, error status return, `throw`/`reject`, fallback, and default branch. Then for every row:

- Is there a test triggering this specific error?
- Is the user/caller-facing outcome verified (not just that an error was thrown)?
- Are ALL error codes/messages tested, not just a sample?

Common miss: `it.each` / `[Theory]` / `@parametrize` testing 3 of 7 error codes.

**Coverage doesn't transfer across boundaries**: A unit test on a dependency (schema, service, utility) does not cover the caller's handling of that dependency's output. Enumerate error paths per caller, not per dependency.

**Silent failures**: If an error branch produces no observable feedback (e.g., toast, error message, redirect, log) — flag as a missing production behavior or verify the silence is intentional.

## Step 3: Check Edge Cases

Reference the `edge-case-heuristics` skill (`skills/edge-case-heuristics/SKILL.md`). Only flag categories relevant to the feature:

- **Boundaries**: n-1, n, n+1 for all limits
- **Empty/zero/null**: Zero items, empty strings, missing params
- **State**: Unmount mid-operation, stale data, concurrent access
- **Interruptions**: Network loss, timeout, session expiry
- **Conditional branches**: Both sides of `||`/`&&`, each fallback step, null at each optional chain level
- **Context variation**: Non-default pathname, locale, role, tenant, environment. Check `beforeEach` for hardcoded mocks no test overrides — the non-default path is untested.

One row per heuristic category in the output. For covered categories, summarize briefly ("all tested" or one sentence). For Partial/Gap categories, expand with specifics of what's missing.

**For conditional branches**: Scan implementation files for branching logic (`if`/`else`, `||`, `&&`, `??`, `?.`, ternaries, `switch`, loops with early returns). Verify both/all branches have test coverage.

## Step 4: Assess Pyramid Distribution

| Level | Expected |
|---|---|
| Unit | 60-70% |
| Integration | 20-30% |
| E2E | 5-10% |

**Classification by mocking strategy**:

| Dependencies | Level |
|--------------|-------|
| **All mocked** or no dependencies | Unit |
| **Mix of mocked and real** | Integration |
| **No mocking** (all real) | E2E |

**Mocking signals:**
- Mocked: `jest.mock()`, `jest.fn()`, `page.route()`
- Real: MSW/`setupServer` (HTTP layer), real DB client, real child components, real context providers
- Test harness: `render()` and `renderHook()` — classify by what's inside (are deps mocked or real?)

**When in doubt**: Choose Integration.

**Examples:**
- Pure function, no deps → Unit
- `render()` + all deps mocked → Unit (component tested in isolation)
- `render()` + real child components or context → Integration (real collaborators)
- `renderHook()` + all deps mocked → Unit (hook tested in isolation)
- API route with all deps `jest.mock()`ed → Unit
- Playwright with `page.route()` → Integration (real browser + mocked API = mix)
- Playwright with no mocking → E2E

Classify per-test, not per-file — a file can contain tests at multiple levels. Anti-pattern: all tests at one level.

## Step 5: Write Review to File

Write results to `specifications/{story-folder}/test-review.md`. Header: `# Test Review: {Story ID} — {Title}` with Date, Story path, Status. Sections separated by `---`:

1. **Test Files Reviewed** — `| File | Type | Tests | Purpose |` with total count. Include ALL test files: unit tests (`__tests__/`), integration tests, AND Playwright tests (`playwright/integration/`, `playwright/e2e/`). Example:
   - `__tests__/components/auth/login.test.tsx` | Integration | 5 | Login form behavior
   - `playwright/e2e/auth-flow.spec.ts` | E2E | 3 | Complete auth user journey
2. **AC → Test Mapping** — `| AC | Unit Test | Integration Test | E2E Test | Verdict |` with `file:line` refs. End with "X of Y ACs have tests. N are partial. K spec discrepancies."
3. **Happy Path Coverage** — `| Unit Under Test | Happy Path Test? | Verdict |`
4. **Error Path Coverage** — `| Error Path | Tested? |` — Yes with `file:line`, or **MISSING**
5. **Edge Cases** — `| Category | Addressed? | Detail |` — one sentence if covered, specifics if gap
6. **Pyramid Distribution** — `| Level | Tests | Percentage | Assessment |`
7. **File-by-File Usefulness** — `### {filename}` per file with: Useful tests (N of N), Suspect tests, Anti-patterns
8. **Recommendations** — `| Priority | File | Issue | Action |` — Action includes what to do and why it matters

**Priority calibration:**
- **CRITICAL**: Untested happy path, untested security/auth path, or an AC with zero coverage
- **HIGH**: Untested error paths, wrong-level tests inflating pyramid, multiple untested branches in a single function
- **MEDIUM**: Name-body mismatch, overlap, single untested edge case, single-level-only coverage for non-critical AC
- **LOW**: Minor edge cases (empty string), cosmetic test improvements, flaky timing patterns
- **E2E gaps**: Missing E2E for the story's primary user journey → **HIGH**. Secondary flows → **MEDIUM**. Never CRITICAL (integration tests provide coverage) or LOW for the primary flow.

## Summary Output

Also summarize in response: 
1. Coverage score: X/Y ACs tested at unit/integration/E2E levels
2. Critical gaps (untested ACs, missing happy paths, spec discrepancies)
3. Edge case gaps (timing, navigation, accessibility, security)
4. Pyramid balance (current vs target percentages for unit/integration/E2E)
5. Prioritized recommendations

**Reminder**: E2E coverage must be reported separately. If you report "0 E2E tests", verify you ran `Glob` with `playwright/**/*.spec.ts` pattern.
