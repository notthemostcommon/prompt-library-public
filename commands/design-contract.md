---
description: Generate task-level design contract from story-level design analysis
globs: "*"
alwaysApply: false
---

"do fas; Generate a design contract for [TASK_ID] as specified in task-[N].md from story [STORY_ID].

INTENTION: Narrow the story-level design analysis into a task-level lookup table of exact Figma values
and their corresponding Tailwind/CSS implementations. This contract eliminates interpretation during
planning and execution — every visual property value is pre-mapped and mechanically validated.

## Prerequisites

Before generating the contract, verify:
1. `design-analysis-[STORY-ID].md` exists in the story folder
2. Task file `task-N.md` exists and describes UI changes
3. Read `.cursor/rules/design-system.mdc` for token mapping rules
4. Read `app/globals.css` to identify existing CSS variables and design tokens
5. Figma MCP is accessible (for filling gaps)

If any prerequisite is missing, STOP and inform the developer.

## Process

### Step 1: Identify Elements in Scope

Read `task-N.md` and list every UI element this task creates or modifies.
For each element, determine its type from this reference:

| Element Type | Required Properties |
|---|---|
| Container/wrapper | padding (all sides), background, border-radius, overflow, width/max-width, gap |
| Text element | font-size, font-weight, line-height, color, letter-spacing (if specified) |
| Button/link | all text properties + padding, background, border, border-radius, min-height (touch target) |
| Image | width, height, object-fit, position, alt behavior |
| Grid/flex layout | direction, columns, gap, alignment, wrapping |

### Step 2: Extract Values from Design Analysis

For each element and each required property:
1. Find the value in `design-analysis-[STORY-ID].md` — copy VERBATIM, do not paraphrase
2. If the value exists, proceed to Step 3
3. If the value is missing or marked `(needs verification)` → go to Figma MCP (Step 2b)

#### Step 2b: Fill Gaps from Figma MCP

For missing values:
1. Use the element's Figma Node ID from the design analysis
2. Call `get_design_context` with that node ID
3. Extract the specific property value from the Figma response
4. Document the source as 'Figma MCP (gap fill)' in the contract

If a value cannot be determined from design analysis OR Figma MCP:
STOP and flag: `❌ Design contract incomplete: [element] [property] could not be determined. Please provide the value or verify Figma node [ID].`

### Step 3: Map Figma Values to Tailwind/CSS

For each Figma value, determine the correct Tailwind class or CSS implementation:

**Spacing** (px ÷ 4 = Tailwind scale):
- 8px → 2, 12px → 3, 16px → 4, 24px → 6, 32px → 8, 40px → 10, 48px → 12, 56px → 14, 80px → 20
- Non-standard values → arbitrary: `px-[22px]` or `px-[1.375rem]`

**Font size**: 12px → text-xs, 14px → text-sm, 16px → text-base, 20px → text-xl, 24px → text-2xl, 36px → text-4xl, 48px → text-5xl, 60px → text-6xl
- Non-standard → arbitrary: `text-[28px]`

**Font weight**: 400 → font-normal, 500 → font-medium, 600 → font-semibold, 700 → font-bold, 800 → font-extrabold

**Border radius**: 4px → rounded, 6px → rounded-md, 8px → rounded-lg, 12px → rounded-xl, 16px → rounded-2xl, 9999px → rounded-full
- Non-standard → arbitrary: `rounded-[2rem]`

**Colors**:
- Check `app/globals.css` for matching CSS variables (search by hex comment)
- If CSS variable exists → use semantic class: `bg-primary`, `text-foreground`, etc.
- If no variable → use arbitrary: `bg-[#f0f5fa]`, `text-[#0a2a44]`
- Standard Tailwind colors are acceptable: `text-gray-600`, `bg-neutral-100`

**Line height**: 16px → leading-4, 20px → leading-5, 24px → leading-6, 28px → leading-7, 32px → leading-8
- Non-standard → arbitrary: `leading-[40px]`

### Step 4: Build Responsive Adaptations

For elements that change across breakpoints, build the responsive table.
Use the 3-tier system from `design-system.mdc`:
- Tier C: xs/sm (default, no prefix)
- Tier B: md/lg (`md:` prefix)
- Tier A: xl/2xl (`xl:` prefix)

Only include rows where a value CHANGES between tiers.

### Step 5: Build Interactive States

For elements with hover, focus, disabled, or other interactive states:
- Extract the state-specific node ID from the design analysis
- Document the property changes per state
- Map to Tailwind state variants: `hover:`, `focus:`, `disabled:`, `active:`

## Gap Handling

If the design analysis is missing values needed for this contract:
1. FIRST: attempt to fill from Figma MCP using the node ID
2. If Figma MCP fails or node ID is missing: STOP and flag to developer
3. NEVER guess or use approximate values
4. NEVER proceed with empty cells in the contract

## Output Format

Create `task-N-design-contract.md` in the story folder:

```markdown
# Design Contract: Task N — [Title]

Source: design-analysis-[STORY-ID].md
Validated: [PENDING — run scripts/validate-design-contract.ts]

## Element Specifications

### [Element Name]
| Property | Figma Value | Tailwind/CSS | Source Node |
|----------|-------------|--------------|-------------|
| [property] | [exact Figma value] | [Tailwind class or CSS] | [node ID] |

## Responsive Adaptations

| Element | Property | Tier C (xs/sm) | Tier B (md/lg) | Tier A (xl/2xl) |
|---------|----------|---------------|----------------|-----------------|
| [element] | [property] | [value] | [value if different] | [value if different] |

## Interactive States

| Element | State | Property | Figma Value | Tailwind/CSS | Source Node |
|---------|-------|----------|-------------|--------------|-------------|
| [element] | hover | [property] | [value] | [class] | [node ID] |
```

## Validation (MANDATORY)

After writing the contract, run:
```
npx tsx scripts/validate-design-contract.ts specifications/[story-folder]/task-N-design-contract.md
```

- If any row FAILS: fix the mapping and re-run until all pass
- Update the `Validated:` line to `✅ PASSED` when the script passes
- Do NOT proceed to planning until validation passes"
