---
name: test-usefulness
description: "Evaluate whether tests provide real value: testing behavior over implementation, catching real bugs, resisting refactoring. Apply when reviewing test quality."
---

# Test Review: Usefulness & Quality

Evaluate whether each test earns its maintenance cost. Suggest improvements for flagged tests.

## The Refactor Test

Ask: "Would renaming an internal function or restructuring internals break this test even though visible behavior is unchanged?" If yes → coupled to implementation. Flag it.

**Apply per-assertion, not per-test.** A test mixing `expect(result).toEqual(activity)` (useful) with `expect(mockSelect).toHaveBeenCalled()` (brittle) is suspect. Read each assertion individually.

### Brittle → Flag

| Testing... | Fix |
|---|---|
| Internal method/function call counts | Assert on observable output or side effects |
| ORM/query-builder chain wiring (`mockSelect`, `mockFrom`, `mockWhere` each called) | Assert on the final payload passed to the DB and the returned result |
| Private field values or internal state | Assert on public API behavior or rendered output |
| DI registrations / hook invocation details | Test through composed behavior |
| DOM structure / element ordering | Query by role, label, or text |
| Specific internal exception types | Assert on status codes or user-facing messages |

### Useful → Keep

CSS classes enforcing design specs (colors, typography, spacing, a11y), user-visible text/behavior/navigation, HTTP status codes and bodies, API contracts, business rule outcomes, user-facing error messages, database query results.

**Boundary data contracts**: Asserting on the payload shape passed to a dependency boundary (DB, API, event emitter) is behavioral — it verifies the data contract. Asserting that intermediate wiring methods were called is not.

> `toHaveClass` for design specs is USEFUL — the class IS the spec. Only flag class assertions with no design or a11y meaning.

**Logging/observability gray area**: Asserting that `console.error` or a logger was called with specific args is implementation-coupled (the output channel could change). However, if the AC explicitly requires failure tracking or if logs are the *only* observable side effect of an error path, testing the log call is acceptable. Prefer asserting on a higher-level observable (error response, metric, alert) when one exists.

## Anti-Patterns

**Tautological** — always passes: `expect(wrapper).toBeDefined()`, `Assert.NotNull(result)`, `assert result is not None`

**Mirror** — re-implements production logic:
```typescript
const expected = items.filter(i => i.active).length
expect(result).toBe(expected) // duplicates the filter
```

**Framework** — tests library behavior, not your code:
```typescript
expect(screen.getByRole('button')).toBeInTheDocument() // ...and nothing else
```

**Overlap** — >2 tests assert same outcome for same input. One per behavior per level suffices. Favor improving overlapping tests to cover untested paths over removing them.

*Detection*: Summarize what each test's assertions verify in one sentence, ignoring the test name. Identical summaries = overlap regardless of different test names.

**Name-body mismatch** — assertions don't verify what the test name claims:
```typescript
// ❌ Name says "network timeout" but assertions only check generic redirect
it('redirects when network timeout occurs', async () => {
  mockFn.mockRejectedValue(new Error('Network timeout'))
  const response = await GET(request)
  expect(response.status).toBe(307) // any error triggers this — not timeout-specific
})
```
Checklist: Does `describe` match precondition? Does `it` match expected outcome? Do assertions verify that specific outcome?

**Hardcoded mock values** — `beforeEach` always sets the same value (e.g., `isAuthenticated: true`) and no test overrides it. The non-default path is untested.

**Wrong-level** — pure function tested through full render/integration:
```typescript
render(<AddressForm />)
await user.type(screen.getByLabelText('City'), "'; DROP TABLE")
// Same bug caught by: expect(validateCity("'; DROP TABLE")).toEqual({ valid: false })
```
Test pure functions directly. Higher-level tests add cost without catching additional bugs.

**E2E-specific anti-patterns:**
- **Timing waits** — `waitForTimeout(500)` is flaky. Use assertion-based waiting (`expect(el).toBeVisible({ timeout })`) or poll for a condition.
- **Third-party selectors** — `[data-sonner-toast]`, `[data-radix-*]`, etc. couple tests to library internals. Query by role, text, or label instead. If no accessible selector exists, that's a production a11y issue, not a test workaround.
- **Conditional guards** — `if (await el.isVisible()) { ... }` silently passes when the element doesn't exist. Use unconditional `expect(el).toBeVisible()` so missing elements fail the test.
- **Data-lookup duplication** — E2E tests re-verifying every row of a lookup table (error code → message) already exhaustively covered at unit/integration level. Keep 1-2 representative E2E flows; unit tests own the data contract.

## Positive Signals

- Behavior in test name: "displays error toast when login fails" not "test case 3"
- Clear AAA: Arrange, Act, Assert (one action, observable outcome)
- Fails for the right reason — a production bug makes exactly this test fail
- Tests one behavior each
- Uses realistic, domain-representative data

## Output Format

Per test file: (1) Useful tests count + summary, (2) Suspect tests with specific concerns, (3) Anti-patterns found, (4) Recommendations for refactoring.
