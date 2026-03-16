---
description:
globs: *
alwaysApply: true
---

META-INSTRUCTION: RULE LOADING (START OF TASK)

Before writing any plan:
1. Read task file to identify target files/extensions
2. Match extensions against `globs` in `<agent_requestable_workspace_rules>`
3. Load ALL matching rules using Read tool
4. Additionally, check if the following quality rules exist in the project's `.cursor/rules/` directory and load them if present:
   - A test strategy rule (e.g. `test-strategy.mdc`) — guides correct test-level decisions during planning
   - A test usefulness skill (e.g. `skills/test-usefulness/SKILL.md`) — flags anti-patterns in planned tests before code is written
   For TypeScript/Next.js projects these are `test-strategy.mdc` and the `test-usefulness` skill (`skills/test-usefulness/SKILL.md`). For other stacks, load the project-equivalent rules/skills if they exist. Skip gracefully if not present.
5. Display in chat response (NOT in the plan file):
   ## Rules Loaded
   - [rule.mdc] - matched [glob] for [target files]

---

"do pla; Create a concise implementation plan for [TASK_ID] as specified in task-[N].md from story [STORY_ID].

INTENTION: Generate a focused, actionable plan that can be immediately executed without extensive refinement.

Create task-[N]-plan.md with this structure:

## 0. Rules Loaded

List every rule file read, by exact path:
- `[relative path to rule file from repo root]` — matched glob `[glob]` for `[target files]`

If no rules matched, write: "No matching rules found."

## 1. Issue

- Clear statement of what needs to be implemented

## 2. Solution

- High-level approach and technical rationale

## 3. Implementation Steps

1. [Specific action with file paths and function names]
2. [Continue with numbered steps...]

## 4. Verification

- Key requirements to validate implementation
- **Test Gate**: If this task includes code changes, all existing unit tests MUST pass before execution is complete

## 5. Rules Compliance Audit

For each rule listed in **§0**, confirm compliance or flag deviation:
- ✅ `[relative path from §0]` — compliant: [brief reason]
- ⚠️ DEVIATION: `[relative path from §0]` — plan uses [X] instead of [required pattern] — **EXECUTION BLOCKED until developer types: `approved: [reason]`**

Any rule in §0 missing from this audit is treated as unreviewed and blocks execution.

Keep the plan concise and actionable. Each step should be specific enough that no creative decisions are needed during execution."

---

META-INSTRUCTION: RULES SUMMARY (END OF TASK)

After completing the plan file, display in chat response (do NOT write to the plan file):

## Rules Applied
**Rules Loaded & Used:**
- [rule] - [how it influenced the plan]

**Rules NOT Loaded:**
- [rule] - [why not relevant to this task]