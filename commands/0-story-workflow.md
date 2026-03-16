---
description: Master agentic story delivery workflow — analysis, decomposition, planning, execution, and review
globs: "*"
alwaysApply: false
---

# Agentic Story Delivery Workflow

Orchestrates the full story lifecycle from analysis through merge-ready review. Run this command at the start of any story to drive the complete delivery cycle.

> **Stack note:** This workflow references TypeScript/Next.js rule/skill names (`test-strategy.mdc`, `skills/test-usefulness/SKILL.md`, `skills/test-review/SKILL.md`). For other stacks, substitute the project's equivalent rules/skills at every step that mentions them.

---

## Intervention Log

Throughout this workflow, the agent must append to `specifications/[story-folder]/workflow-interventions.md` whenever the developer provides a correction, clarification, or any response that contains substantive content beyond a bare approval token.

**Do not log:** bare approval tokens (`approved: analysis`, `approved: all plans`, etc.)
**Do log:** any developer message that corrects the agent, adds missing context, overrides a decision, or identifies something the agent missed.

Each entry uses this format:

```md
## [Phase N / Task N] — [short title]
**Category:** missed-requirement | wrong-pattern | false-positive | clarification | override | other
**Agent missed:** [what the agent got wrong or failed to catch]
**Developer correction:** [verbatim or close paraphrase of what the developer said]
```

Create the file on first intervention. If no interventions occur, do not create the file.

---

## Before You Start

Verify these inputs are available:
- `specifications/[story-folder]/story.md` exists and contains acceptance criteria
- Figma link is present in story (if design work is involved)
- Story folder follows convention: `specifications/[STORY-ID]-[slug]/`

---

## Phase 1 — Story Analysis

**Purpose:** Understand the story deeply before any task breakdown or planning.

### 1a. BDD Analysis

Read and apply the `bdd-story-analysis` skill (`skills/bdd-story-analysis/SKILL.md`). Append BDD scenarios and test distribution table directly to `story.md`. Do not create a separate file.

Output written to: `specifications/[story-folder]/story.md` (appended)

### 1b. Design Analysis *(skip if no Figma link in story)*

Run `analyze-design-requirements.md`. Don't guess, confirm or ask. Agent stops if MCP/variables unavailable — fix setup first. Output must include "Design Analysis Sources" at top.

**Completeness Self-Audit:** Before presenting to Gate 1, verify the design analysis covers every component with: Figma node ID, dimensions per breakpoint tier, padding (all sides), typography per text element (size, weight, line-height, color), background/border colors with hex, border-radius, and interactive state node IDs. Any empty cell must be filled from Figma MCP or flagged to the developer. The "Design Analysis Sources" table must be present and accurate.

Output written to: `specifications/[story-folder]/design-analysis-[STORY-ID].md`

### 1c. Test Strategy Audit

Apply `test-strategy.mdc` to the BDD test distribution table just written in `story.md`.

Do not create a file. In the chat response only, flag:
- Any AC with no planned test level
- Any pure function being tested at integration/E2E instead of unit
- Any UI journey being tested only at unit level
- Missing error path coverage plans
- Missing happy path coverage plans

Append a `## Test Strategy Audit` section to `story.md` summarising these flags.

---

### ▶ GATE 1 — Developer sign-off on analysis

Present a summary table to the developer:

```
Analysis complete for [STORY-ID]:
  ✅ BDD scenarios written: [N] scenarios covering [M] ACs
  ✅ Design analysis: [present — N components, all with node IDs / not applicable]
  ✅ Design completeness: [all values sourced from Figma variables / N values need verification]
  ✅ Test strategy audit: [N flags / no flags]

Review specifications/[story-folder]/story.md then reply:
  approved: analysis
```

**Do not proceed to Phase 2 until the developer replies `approved: analysis`.**

---

## Phase 2 — Decomposition

**Purpose:** Break the story into independently executable tasks.

Run command `1-decomposition.md` for the story.

This produces:
- `task-01.md` ... `task-N.md`
- `task-N-review.md` (final review task, always created last)

Each task file must include a `parallel-group` field (see `1-decomposition.md` for guidance). Tasks with no inter-dependencies share the same group label (A, B, C...). Dependent tasks get their own group or reference the group they must follow.

After creating all task files, output an **informational** task breakdown summary (no gate, auto-proceed):

```
Task breakdown for [STORY-ID]:
  task-01: [title] — group: A (sequential)
  task-02: [title] — group: B (sequential, depends on task-01)
  task-03: [title] — group: C (parallel with task-04)
  task-04: [title] — group: C (parallel with task-03)
  task-05-review: final review

Proceeding to Phase 3 — Planning...
```

---

## Phase 3 — Planning (per task, sequential)

**Purpose:** Produce an approved implementation plan for every task before any code is written.

Repeat the following for **each** `task-N.md` (excluding the review task):

### Per-task planning loop

1. **Research** — run command `3-research.md` for the task. Understand existing patterns and integration points.
2. **Design Contract** *(skip if task has no UI file changes — no .tsx/.jsx/.css in scope)* — run command `design-contract.md` for the task. This produces `task-N-design-contract.md` by extracting exact Figma values and Tailwind mappings for elements this task touches. The agent runs `npx tsx scripts/validate-design-contract.ts` to mechanically verify all mappings. Contract must PASS validation before proceeding.
3. **Plan** — run command `2-plan-task.md` for the task. This produces `task-N-plan.md`.
   - `test-strategy.mdc` and the `test-usefulness` skill (`skills/test-usefulness/SKILL.md`) are loaded during this step (see `2-plan-task.md` META-INSTRUCTION).
   - If `task-N-design-contract.md` exists, the plan's Implementation Steps MUST reference it for all visual property values. Do NOT restate values in prose — say "per design contract" and reference the contract file.
4. **Validate** — run command `4-review-update-plan.md`. Confirm no creative decisions remain. If a design contract exists, confirm every CSS class in the plan has a corresponding contract row.

After all plans are produced, present the **bulk plan summary** to the developer:

```
All plans ready for [STORY-ID]:
  task-01-plan.md — [one-line summary]
  task-02-plan.md — [one-line summary]
  task-03-plan.md — [one-line summary]
  task-04-plan.md — [one-line summary]

Review each plan file. These are the execution contracts.
To approve all and begin execution, reply:
  approved: all plans
```

---

### ▶ GATE 3 — Developer sign-off on all plans

**Do not proceed to Phase 4 until the developer replies `approved: all plans`.**

---

## Phase 4 — Execution + Agentic Review Loop (per task)

**Purpose:** Execute each approved plan, review, and loop until all Tier 1 issues are clear before moving to the next task.

### Parallelization guide

At the start of Phase 4, output the full execution plan. For every task that can run in parallel with another, generate a **complete copy-paste instruction block** so the developer can launch a second agent without writing anything themselves.

Example output format:

```
═══════════════════════════════════════════════════
  PHASE 4 — EXECUTION PLAN  [STORY-ID]
═══════════════════════════════════════════════════
Sequential:
  → task-01 (group A)
  → task-02 (group B, follows A)

Parallel group C (run simultaneously after task-02):
  → task-03  ← this agent will handle this
  → task-04  ← open a second Cursor agent and paste the block below

─────────────────────────────────────────────────
  COPY THIS BLOCK INTO A NEW CURSOR AGENT (Cmd+T)
─────────────────────────────────────────────────
do exe; Execute task-04 for story [STORY-ID].

Story folder:  specifications/[story-folder]/
Task file:     specifications/[story-folder]/task-04.md
Plan file:     specifications/[story-folder]/task-04-plan.md

Follow task-04-plan.md exactly — no deviations.
After execution:
  1. Run: npm test
  2. Run code review: code-review against task-04
  3. Run test review: test-review against specifications/[story-folder]/
If Tier 1 issues are found, write task-04-fix-1-plan.md and stop.
Do not self-fix. Report: "Gate 4 required for task-04: [summary of issues]"
─────────────────────────────────────────────────

This agent will now execute task-03. Proceeding...
═══════════════════════════════════════════════════
```

Generate one copy-paste block per parallel task. This agent then immediately proceeds with its own assigned task.

### Per-task execution loop

For each task, in sequence (or concurrently per group):

#### Step 1 — Execute
Run command `5-execution.md` for the task. Follow `task-N-plan.md` exactly.
If `task-N-design-contract.md` exists, the agent MUST have it open during execution and use it as the lookup table for all visual property values. Any value not in the contract that the agent needs during execution → STOP and return to planning (deviation from plan).

#### Step 2 — Run tests
```
npm test -- --testPathPattern=[relevant pattern]
```
If tests fail, fix before proceeding to reviews. A failing test suite is a Tier 1 issue by definition.

#### Step 3 — Agentic reviews

Run all applicable reviews:

| Review | Command | Always? |
|---|---|---|
| Code review | `code-review.md` | Yes |
| Test usefulness | apply `test-usefulness` skill (`skills/test-usefulness/SKILL.md`) to new/modified test files | Yes |
| Test coverage | `test-review.md` | Yes |
| Design review | `design-review.md` | Only if Figma link present in story |

#### Step 4 — Tier 1 triage

**Blocking (Tier 1) criteria by review type:**

| Review | Blocking issues |
|---|---|
| Code review | Any `❌` (MUST FIX) |
| Test coverage | Any CRITICAL gap (untested happy path, zero AC coverage, spec discrepancy) |
| Test usefulness | Tautological or mirror tests that would mask real bugs |
| Design review | `🎨 [HIGH]`, `🎯 [CRITICAL/HIGH]`, `♿ [CRITICAL/HIGH]` |
| Design contract | Any implemented CSS value that contradicts `task-N-design-contract.md` |

**If zero Tier 1 issues:** auto-proceed to next task.

**If Tier 1 issues found:** trigger the fix loop below.

---

### ▶ GATE 4 — Fix plan approval (per violation set)

The agent produces `task-N-fix-M-plan.md` containing:
- List of Tier 1 violations being addressed
- Minimal implementation steps to resolve each
- Rules compliance audit for the fix

Present to developer:

```
Tier 1 issues found after task-N execution.
Fix plan written to: task-N-fix-M-plan.md

Issues:
  ❌ [description] — [file:line]
  ❌ [description] — [file:line]

Reply to approve and execute the fix:
  approved: fix-N-M
```

**Do not execute the fix until the developer replies `approved: fix-N-M`.**

After fix execution: re-run tests → re-run reviews → evaluate Tier 1 again → loop until clear.

Once clear: **auto-proceed to next task.**

---

## Phase 5 — Final Story Review

**Purpose:** Story-level quality gate across all implemented tasks combined.

### 5a. Story-level code review
Run `code-review.md` against `specifications/[story-folder]/story.md`. This triggers git log aggregation and full diff review.

### 5b. Design review *(skip if no Figma link)*
Run `design-review.md` for the full story.

### 5c. Test coverage review
Run `test-review.md` for the full story.
Output written to: `specifications/[story-folder]/test-review.md`

---

### ▶ GATE 5 — Final developer sign-off

```
Final story review complete for [STORY-ID]:
  Code review:   [Ready to merge / Requires changes]
  Design review: [Ready to merge / Requires changes / N/A]
  Test review:   [X/Y ACs covered, N critical gaps]

All Tier 1 issues resolved: [Yes / No — list remaining]
All tests passing: [Yes / No]

To approve for merge, reply:
  approved: merge
```

**Do not mark story complete until the developer replies `approved: merge`.**

---

## Approval Token Reference

| Token | Phase | Meaning |
|---|---|---|
| `approved: analysis` | Gate 1 | BDD, design analysis, and test strategy audit are acceptable |
| `approved: all plans` | Gate 3 | All `task-N-plan.md` files are acceptable execution contracts |
| `approved: fix-N-M` | Gate 4 | Fix plan for task N, fix set M is approved to execute |
| `approved: merge` | Gate 5 | Story is merge-ready |

---

## File Naming Conventions

| File | Convention |
|---|---|
| Task requirements | `task-01.md`, `task-02.md` (zero-padded) |
| Final review task | `task-N-review.md` |
| Task plans | `task-01-plan.md`, `task-02-plan.md` |
| Fix plans | `task-N-fix-M-plan.md` (N=task number, M=fix iteration) |
| Design analysis | `design-analysis-[STORY-ID].md` |
| Design contracts | `task-01-design-contract.md`, `task-02-design-contract.md` |
| Test review | `test-review.md` |
| Intervention log | `workflow-interventions.md` (created on first intervention only) |

All files created in `specifications/[story-folder]/`.
