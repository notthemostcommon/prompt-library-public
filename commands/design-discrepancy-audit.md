---
description: Pre-implementation Figma design audit to surface discrepancies and gaps for designers
globs: "*"
alwaysApply: false
---

"do fas; Perform a pre-implementation design discrepancy audit on the Figma design at [FIGMA_LINK] for story [STORY_ID].

INTENTION: Systematically analyze Figma designs BEFORE implementation to surface internal
inconsistencies, gaps, and deviations from the design system. This prevents costly rework by
letting designers resolve ambiguities before any code is written.

This is NOT a design analysis for implementation specs — it produces a discrepancy report
for the design team. See `specifications/figma-home-vs-journey-discrepancies.md` for an
example of the expected output format and level of detail.

## Prerequisites

Before starting the audit, verify:
1. Figma MCP is accessible (test with `get_metadata`)
2. Read `.cursor/rules/design-system.mdc` for the canonical design token and breakpoint reference
3. Read `app/globals.css` to identify existing CSS variables

If any prerequisite fails, STOP and inform the developer.

## Process

### Step 1: Discover All Frames

Call `get_metadata` on the provided Figma node to get the full page structure.

Extract every frame that represents a breakpoint variant or interactive state:
- Group by screen/component name and breakpoint (xxl, xl, lg, md, sm, xs)
- Identify interactive state variants (hover, selected, disabled, open, closed)
- Log the frame inventory as a table: name, node ID, dimensions

Flag immediately:
- Missing breakpoint variants (e.g., screen exists at xxl but not at sm)
- Missing interactive states (e.g., hover only at one breakpoint)
- Inconsistent frame dimensions vs expected breakpoint widths

### Step 2: Extract Key Properties Across Breakpoints

For EACH unique screen/component, call `get_design_context` at every breakpoint variant.

Extract and tabulate these properties across all breakpoints:

**Layout:**
- Page/container side padding (gutter)
- Content area width
- Grid layout (columns, gap)
- Element positioning

**Typography:**
- Heading font sizes, weights, and line heights
- Body text properties
- Compare against design system typography scale (3-tier system)

**Spacing:**
- Gaps between major sections
- Padding on cards/containers
- Check if values use design tokens or hardcoded pixels

**Element Sizing:**
- Card/selection widths and heights
- Button widths and heights
- Image dimensions

**Interactive Elements:**
- Side panels, drawers, modals — presence and dimensions per breakpoint
- Button layout (justify-between vs side-by-side, auto vs stretch)

### Step 3: Cross-Breakpoint Consistency Audit

For each property extracted in Step 2, check:

1. **Monotonic scaling**: Do element sizes scale logically with viewport?
   Flag: xs values > sm values, or lg values > xxl values (size inversions)

2. **Design system compliance**: Do values match the documented token set?
   Flag: Non-standard spacing values, font sizes not in the typography scale,
   padding that doesn't match the 3-tier gutter system

3. **Token binding**: Are Figma variables used, or are values hardcoded?
   Flag: Hardcoded pixel values where a spacing/color token should be used

4. **Tier consistency**: Within each tier (A: xl/2xl, B: md/lg, C: xs/sm),
   are values consistent between the two breakpoints?
   Flag: Values that differ within the same tier

5. **Structural consistency**: Is the component hierarchy/naming the same
   across breakpoints, or does it change?
   Flag: Different layer names, missing child elements, restructured hierarchy

### Step 4: Interactive State Audit

For each interactive state variant found in Step 1:

1. Is the state designed for ALL breakpoints, or only one?
2. Does the state visual (e.g., selection indicator, hover highlight) work at
   all element sizes across breakpoints?
3. Are there any states referenced in the UI (e.g., a 'Need help?' button) that
   have no corresponding state design at certain breakpoints?

### Step 5: Compare Against Existing Implementation (if applicable)

If the feature being designed already has an implementation:
1. Search the codebase for existing components
2. Identify where the new Figma design differs from current code
3. Note whether differences are intentional redesigns or oversights
4. If existing implementation handles a case the Figma doesn't show
   (e.g., responsive behavior), note this as a gap

### Step 6: Check for Text/Content Issues

Scan text content across breakpoints for:
- Typos or extra whitespace (double spaces, trailing spaces)
- Text that doesn't fit its container at smaller breakpoints
- Inconsistent capitalization across the same labels

## Output Format

Create `specifications/figma-[feature-name]-discrepancies.md`:

```markdown
# Figma [Feature Name] Design Discrepancies

**Figma Source:** [link]
**Design System Reference:** `design-system.mdc` (3-tier system)
**Frames analyzed:** [count] frames across [N] screens × [M] breakpoints + interactive states

---

## N. [Discrepancy Title]

[Brief description of the issue]

| Breakpoint | [Property] | [Expected/Comparison] | Match? |
| ---------- | ---------- | --------------------- | ------ |
| xxl        | [value]    | [expected]            | ✅/❌  |
| ...        | ...        | ...                   | ...    |

**Decision needed:** [Clear question for the design team]

---

[Repeat for each discrepancy]

---

## Summary: decisions needed

[Numbered list referencing each discrepancy]

### Blocking items (cannot implement without resolution)

- [Items that prevent implementation entirely]

### High-priority items (will cause AI implementation confusion)

- [Items that will lead to incorrect or inconsistent code]
```

## Severity Classification

Classify each discrepancy:

**Blocking** — Cannot implement without designer input:
- Missing breakpoint designs for key screens
- Undefined responsive behavior for interactive elements
- Values that would cause layout breakage (e.g., huge text on tiny viewport)

**High priority** — Will cause implementation confusion:
- Non-monotonic element sizing across breakpoints
- Non-standard tokens that break the design system
- Padding/spacing mismatches with existing pages
- Missing interactive state designs

**Medium priority** — Should be fixed but won't block:
- Hardcoded values where tokens should be used
- Inconsistent border/shadow styling
- Minor structural naming differences

**Low priority** — Cosmetic or nice-to-have:
- Text typos
- Gradient width mismatches
- Minor padding differences within a tier

## Rules

- NEVER guess what the designer intended — flag and ask
- Compare against `design-system.mdc` as the canonical reference
- Include Figma node IDs for every discrepancy so designers can locate issues
- Present findings neutrally — 'Decision needed' not 'This is wrong'
- Group related discrepancies into single items when they share a root cause
- If an existing implementation already handles a gap, note it rather than flagging as blocking"
