---
name: troubleshooting
description: "Common error solutions and debugging approaches. Apply when debugging errors or investigating failures."
---

# AI Agent Troubleshooting Rules

> **Core Principle:** When the user says something worked before, BELIEVE THEM. Start from the assumption that recent changes introduced the problem.

---

## The Universal 3-Step Protocol

### 1️⃣ **RESET THE ENVIRONMENT**

Most issues are environmental, not logical.

**Before investigating ANY issue:**

- Kill stale processes (dev servers, background jobs, databases)
- Clear all caches (build artifacts, dependency caches, test results)
- Verify clean git state
- Fresh build/install

**Red Flags for Environment Issues:**

- 🚩 Works sometimes, fails other times
- 🚩 Errors don't match current code/files
- 🚩 "Already in use" / "Cannot find" errors
- 🚩 Behavior changed without code changes

**Action:** Clean environment BEFORE debugging logic.

---

### 2️⃣ **ESCAPE THE COMPLEXITY TRAP**

Solutions should get simpler, not more complex.

**You're in a death spiral if:**

1. 🚩 Solutions getting **longer/more complex** (not simpler)
2. 🚩 On **attempt 3+** without progress
3. 🚩 **Working around** problems your solution created
4. 🚩 **Manually implementing** what tools/frameworks should handle
5. 🚩 Can't explain solution in **one clear sentence**

**Recovery Protocol:**

```
STOP → Don't make it worse
  ↓
RESET → State the ROOT problem in one sentence
  ↓
SEARCH → "tool/language + problem + best practice"
  ↓
SIMPLIFY → Use standard patterns, trust the tools
  ↓
STUCK? → External help after 3 attempts
```

**Critical Questions:**

- ❓ "What's the **actual problem** vs problems my solution created?"
- ❓ "Is there a **standard pattern/idiom** for this?"
- ❓ "Am I **trusting the tools** to do their job?"
- ❓ "Would an experienced developer suggest **something simpler**?"

> **Universal Truth:** Complex problems often have simple solutions. Standard patterns exist for common problems. Step back beats push forward when stuck.

---

### 3️⃣ **BE METHODICAL**

Work from simple → complex, not the reverse.

```
Problem reported
  ↓
Did YOU just change something?
  YES → Your changes likely broke it
  NO → Continue
  ↓
Clean environment (Step 1)
  ↓
Rebuild/restart fresh
  ↓
Still broken?
  ↓
Compare to last working state
  ↓
Identify what changed
  ↓
Isolate the specific change
  ↓
Still stuck? Ask for clarification
```

---

## Test Verification Protocol

When verifying test results, you must only make claims about outcomes you directly observed.

### The Rule: Only Claim Observed Outcomes

Don't claim test outcomes (pass/fail/skip) you didn't directly observe. Don't calculate aggregate statistics when some runs have incomplete data.

### Incomplete Output Handling

**When you have incomplete output:**

Distinguish between what you observed and what you don't know:
- ✅ State what the output showed
- ✅ Note anomalies or patterns for investigation
- ✅ Acknowledge missing data explicitly
- ❌ Fill gaps with assumptions about unobserved outcomes
- ❌ Calculate aggregate statistics that include incomplete runs

**Action steps:**
1. State what you observed from the incomplete output
2. Acknowledge what's unknown
3. Either re-run with complete capture, or proceed with only the known data

### Best Practices for Verification

```bash
# DO: Capture complete output
run_tests --reporter=list 2>&1 | tee output.log

# DO: Save each iteration for multi-run verification
for i in 1 2 3; do
  echo "=== Run $i ===" | tee -a all-runs.log
  run_tests 2>&1 | tee -a all-runs.log
done

# AVOID: Truncating verification output (use for quick checks, not proof)
run_tests | tail -2  # Useful for seeing if error occurred, not for claiming success
```

**Core principle:** Only claim outcomes you directly observed. If data is incomplete, acknowledge the gap.

---

## Universal Principles

### ❌ DON'T

**Argue with the user's baseline**  
→ They know their system. If they say it worked, it did.

**Jump to complex architectural theories**  
→ Start simple: environment, cache, processes, dependencies

**Continue down a failing path**  
→ 3 attempts without progress = wrong approach

**Add complexity to fix complexity**  
→ If your fix is longer than the original, step back

**Implement what tools should handle**  
→ Frameworks/libraries exist to solve common problems

### ✅ DO

**Start with environment/cache first**  
→ Most issues resolve here (stale processes, caches, dependencies)

**Trust the tools**  
→ Use standard patterns/idioms before custom logic

**Communicate what you're checking**  
→ "Verifying clean environment..." builds trust

**Stop early when stuck**  
→ Ask for help after 3 failed attempts

**Be methodical, not defensive**  
→ Simple → complex, accept being wrong quickly

---

## When to Ask for Help

After you've:

1. ✅ Cleaned environment completely
2. ✅ Rebuilt/restarted from scratch
3. ✅ Attempted 3 different approaches
4. ✅ Issue persists consistently

Then: "Issue persists in clean environment after [X approaches]. May I get clarification on [specific detail]?"

---

## Remember

**The user is your partner, not your adversary.**

When they report an issue:

- They're usually right
- They know their codebase
- They want to move forward, not debate

Your job is to help, not to be right.

---

## Codebase-Specific Examples

> **Note:** These are examples for common scenarios. Adapt the principles above to your specific stack.

<details>
<summary><b>Example: Test Failures (Jest, Playwright, Pytest, etc.)</b></summary>

**Environment Reset:**

```bash
# Kill dev servers on test ports
lsof -ti:3000 | xargs kill -9 2>/dev/null  # Adjust port

# Clear test artifacts
rm -rf test-results playwright-report coverage .pytest_cache

# Clear framework caches
rm -rf .next dist build __pycache__ node_modules/.cache

# Fresh build
npm run build  # or equivalent
```

**Common Issues:**

- Tests hitting stale dev server instead of fresh build
- Cached test results or coverage reports
- Background processes from previous runs

**Standard Pattern:** Let test frameworks manage their lifecycle (don't manually start/stop servers)

</details>

<details>
<summary><b>Example: Build/Deployment Failures</b></summary>

**Environment Reset:**

```bash
# Clear build outputs
rm -rf dist build out target .next

# Clear dependency caches
rm -rf node_modules/.cache .gradle/cache pip-cache

# Fresh install
npm ci  # or pip install --force-reinstall, gradle clean, etc.
```

**Common Issues:**

- Stale build artifacts
- Dependency version mismatches
- Cached environment variables

**Standard Pattern:** Use CI/build tool features (caching, artifacts) rather than custom scripts

</details>

<details>
<summary><b>Example: API/Service Integration Issues</b></summary>

**Environment Reset:**

```bash
# Restart dependent services
docker-compose restart  # or equivalent

# Clear application caches
redis-cli FLUSHALL  # if using Redis

# Verify service health
curl http://localhost:PORT/health  # check endpoints
```

**Common Issues:**

- Services in bad state from previous debugging
- Cached responses or stale connections
- Configuration/secrets not loaded

**Standard Pattern:** Use service discovery, health checks, retry logic from libraries

</details>

<details>
<summary><b>Example: Database/Migration Issues</b></summary>

**Environment Reset:**

```bash
# Reset to known state
db:rollback && db:migrate  # Rails/ActiveRecord
python manage.py migrate --fake-initial  # Django
flyway clean && flyway migrate  # Flyway

# Or fresh database
dropdb mydb && createdb mydb && db:migrate
```

**Common Issues:**

- Migrations partially applied
- Schema out of sync with models
- Test data in wrong state

**Standard Pattern:** Use ORM/migration tool features, not custom SQL scripts

</details>

---

## THIS PROJECT: web-app (Next.js + Playwright)

**Quick Environment Reset:**

```bash
lsof -ti:3000 | xargs kill -9 2>/dev/null
rm -rf .next node_modules/.cache playwright-report test-results coverage
npm ci
```

**Common Issues:**

- `.next` cache after significant changes
- Playwright webServer managing server lifecycle (don't manually start/stop)
- MSW mocking enabled via `NEXT_PUBLIC_API_MOCKING=enabled`

**When Tests Hang:**

- Check for HTML reporter in CI (should use 'list' reporter)
- Verify cleanup functions use `trap` for guaranteed execution
- Trust Playwright's webServer config to manage server lifecycle
