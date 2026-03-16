---
description: Analyze Figma design files to extract implementation requirements
globs: *
alwaysApply: false
---
"do fa; Analyze the Figma design file at [FIGMA_LINK] using Figma MCP servers to extract comprehensive implementation requirements.

Follow the rules in ../rules/design-system.mdc

INTENTION: Transform visual designs into actionable technical specifications by systematically 
analyzing design tokens, components, states, interactions, and responsive behavior. This analysis 
will serve as the foundation for accurate implementation that matches design intent.

## CRITICAL: Don't guess, confirm or ask

- **Tokens:** Use `get_variable_defs` for colors, spacing, typography, borders, shadows. Do NOT infer from screenshots or `get_design_context`.
- **If `get_variable_defs` fails or returns empty:** STOP. Prompt: "❌ Figma variable extraction failed. Please ensure Figma MCP is connected and the design uses Figma variables. I cannot infer tokens from screenshots."
- **Layout:** `get_design_context` OK for hierarchy/layout only. Token-like values in that output → document as "From screenshot" in Sources, flag for verification.

Using the Figma MCP server connection, perform the following analysis:

0. VISUAL REFERENCE (MANDATORY — get_screenshot)
   - Call `get_screenshot` with the provided fileKey and nodeId
   - This screenshot is the persistent visual reference throughout the entire analysis
   - Keep it accessible for cross-checking component inventory, states, and responsive behavior
   - If the design has multiple breakpoint frames, capture screenshots of each

1. DESIGN TOKENS EXTRACTION (MANDATORY FIRST — get_variable_defs)
   - 1a: Call `get_variable_defs` for node(s)/page.
   - 1b: If fails or empty → STOP, prompt developer (see CRITICAL above).
   - 1c: If design has no variables but tokens needed → STOP, prompt to add variables or provide canonical analysis.
   - 1d: Only after success, proceed. Do not use get_design_context for token values.
   
   **COLOR CONVERSION (hex → OKLCH)**: Figma exports colors as hex. Our design system uses OKLCH.
   For each color variable, run: `node scripts/convert-hex-to-oklch.mjs #HEXVALUE`
   Use the OKLCH output in the design analysis and for globals.css. Document both hex (Figma source) and OKLCH (implementation) in the tokens.
   
   For each variable category, document:
   - **Variable Name** (as defined in Figma, e.g., 'color/primary/500', 'spacing/lg')
   - **Value** (hex from Figma; OKLCH from conversion for colors—use script above)
   - **Scope** (where it's used: colors, typography, spacing, etc.)
   - **Semantic Meaning** (purpose/usage guidelines)
   
   Extract and organize:
   - Color variables (primary, secondary, semantic colors, states)
   - Typography variables (font families, sizes, weights, line heights, letter spacing)
   - Spacing variables (margins, paddings, gaps, named by scale)
   - Border radius variables
   - Shadow/elevation variables
   - Animation/transition timing variables
   - Breakpoint variables for responsive design
   - Opacity/alpha variables
   - Z-index layering variables

2. COMPONENT INVENTORY (Use get_code_connect_map when available)
   
   **Handling large/complex designs**: If `get_design_context` returns truncated or incomplete data:
   1. Call `get_metadata(fileKey, nodeId)` to get the high-level node map and child structure
   2. Identify the specific child node IDs from the metadata
   3. Re-fetch individual child nodes with `get_design_context(fileKey, childNodeId)`
   4. Repeat for any sub-nodes that are still truncated
   
   - Use get_code_connect_map to identify components already mapped to code
   - List all reusable components in the design
   - Document component hierarchy and composition
   - Identify atomic components vs. composite components
   - Note component variants and their naming conventions
   - Document component properties and their accepted values
   - For each component, note which design tokens/variables it uses
   - Cross-reference with existing codebase components (via Code Connect)
   
   **CRITICAL: Check for applicable shadcn/ui components**:
   - For EACH component in the design, evaluate if a shadcn component exists
   - Common shadcn components: Card, Button, Input, Textarea, Form, Dialog, Drawer, Select, Skeleton, etc.
   - **DEFAULT TO SHADCN**: If a shadcn component exists, plan to use it with className customization
   - Document: "Shadcn Component Mapping" section for each component
   - Only skip shadcn if there's a technical incompatibility (rare)

3. COMPONENT STATES ANALYSIS
   - Default state
   - Hover state
   - Active/pressed state
   - Focused state
   - Disabled state
   - Loading state
   - Error state
   - Success state
   - Document state transitions and triggers

4. INTERACTION BEHAVIOR
   - Click/tap interactions
   - Drag and drop behavior
   - Form interactions and validation patterns
   - Navigation patterns
   - Modal and overlay behavior
   - Scroll interactions
   - Animation sequences and micro-interactions
   - Gesture support (swipe, pinch, etc.)

5. RESPONSIVE DESIGN ANALYSIS
   - Mobile breakpoint behavior (< 768px)
   - Tablet breakpoint behavior (768px - 1024px)
   - Desktop breakpoint behavior (> 1024px)
   - Layout changes across breakpoints
   - Component adaptations for different screen sizes
   - Touch vs. mouse interaction considerations
   - Orientation handling (portrait/landscape)

6. ACCESSIBILITY CONSIDERATIONS
   - Color contrast ratios
   - Text sizing and readability
   - Focus indicators
   - ARIA labels and roles (where implied by design)
   - Touch target sizes

OUTPUT FORMAT:
Create a design-analysis-[DATE].md file that includes:

## Design Analysis Sources (REQUIRED)

| Source | Contents |
|--------|----------|
| From Figma variables (get_variable_defs) | [list or "none"] |
| From screenshot/get_design_context | [list or "none"] |
| From existing design-analysis-*.md | [list or "none"] |

Inferred token values → list with "(needs verification)". If tokens required and get_variable_defs unavailable → file must not exist (you stopped).

## Design Overview
- Brief description of the design
- Key user flows
- Design system references

## Design Tokens (Extracted from Figma Variables)
Document the actual Figma variable names for reusability:

```json
{
  "colors": {
    "figmaVariableName": "color/primary/500",
    "figmaValue": "#3B82F6",
    "value": "oklch(0.585 0.193 264.376)",
    "cssVarName": "--color-primary-500",
    "usage": "Primary action buttons, links"
  },
  "typography": {
    "figmaVariableName": "typography/heading/h1/size",
    "value": "32px",
    "cssVarName": "--text-h1-size",
    "usage": "Page main headings"
  },
  "spacing": {
    "figmaVariableName": "spacing/lg",
    "value": "24px",
    "cssVarName": "--spacing-lg",
    "usage": "Large gaps between sections"
  },
  "borders": {
    "figmaVariableName": "border/radius/md",
    "value": "8px",
    "cssVarName": "--border-radius-md",
    "usage": "Cards, buttons"
  },
  "shadows": {
    "figmaVariableName": "shadow/elevation/2",
    "value": "0 4px 6px rgba(0,0,0,0.1)",
    "cssVarName": "--shadow-elevation-2",
    "usage": "Elevated cards"
  },
  "animations": {
    "figmaVariableName": "animation/duration/normal",
    "value": "200ms",
    "cssVarName": "--animation-duration-normal",
    "usage": "Standard transitions"
  },
  "breakpoints": {
    "figmaVariableName": "breakpoint/tablet",
    "value": "768px",
    "cssVarName": "--breakpoint-tablet",
    "usage": "Tablet layout switch"
  }
}
```

### Color Token Format
For colors: always convert Figma hex to OKLCH using `node scripts/convert-hex-to-oklch.mjs #HEXVALUE`.
Document `figmaValue` (hex) and `value` (OKLCH) so both source and implementation are traceable.

### Unit Format (rem)
Spacing, dimensions, typography sizes in globals.css use rem (16px base). When adding new px tokens from Figma, run `node scripts/convert-px-to-rem.mjs app/globals.css --write` or convert manually (px ÷ 16 = rem). Breakpoints stay in px.

### Variable Naming Convention
Document the pattern used in Figma variables:
- Naming structure: `{category}/{subcategory}/{variant}`
- Examples: `color/primary/500`, `spacing/md`, `typography/body/regular`
- This naming should be preserved in code implementation for consistency

## Component Library
For each component, document which design tokens it uses for future reusability.

### [Component Name]
- **Purpose**: 
- **Variants**: 
- **States**: 
- **Props/Properties**: 
- **Composition**: 
- **Design Tokens Used**:
  - Colors: (list Figma variable names, e.g., `color/primary/500`)
  - Typography: (list Figma variable names, e.g., `typography/body/medium`)
  - Spacing: (list Figma variable names, e.g., `spacing/md`, `spacing/lg`)
  - Borders: (list Figma variable names, e.g., `border/radius/sm`)
  - Shadows: (list Figma variable names, e.g., `shadow/elevation/1`)
  - Animations: (list Figma variable names, e.g., `animation/duration/fast`)
- **Responsive Behavior**: 
- **Code Connect**: (if component already exists in codebase, note the mapping)
- **Shadcn Component Mapping**: 
  - Check if shadcn component exists (Card, Button, Skeleton, etc.)
  - If YES: Document which shadcn component(s) to use and how to customize
  - If NO: Document why custom implementation is needed
  - Installation required: Note if `npx shadcn@latest add [component]` is needed

Example:
### Button Component
- **Purpose**: Primary call-to-action element
- **Variants**: Primary, Secondary, Tertiary, Destructive
- **States**: Default, Hover, Active, Disabled, Loading
- **Props/Properties**: size (sm, md, lg), variant, disabled, loading
- **Composition**: Icon (optional) + Label + Loading Spinner (conditional)
- **Design Tokens Used**:
  - Colors: `color/primary/500`, `color/primary/600`, `color/neutral/white`
  - Typography: `typography/button/medium`, `typography/button/large`
  - Spacing: `spacing/xs` (padding), `spacing/sm` (icon gap)
  - Borders: `border/radius/md`
  - Shadows: `shadow/elevation/1` (hover state)
  - Animations: `animation/duration/fast` (state transitions)
- **Responsive Behavior**: Touch targets expand to 44px min on mobile
- **Code Connect**: Mapped to `components/Button.tsx`

FILE ORGANIZATION:
- Create analysis file in the same directory as the related story/task
- Use naming convention: design-analysis-[STORY_ID].md
- Link the analysis file in the relevant story or task file

## COMPLETENESS SELF-AUDIT (MANDATORY — run before presenting to Gate 1)

Before finalizing the design analysis, verify each component in the Component Library section passes this rubric. Any ❌ must be resolved (go back to Figma MCP) or explicitly flagged to the developer.

**Per-component checklist:**

| Requirement | Check |
|---|---|
| Figma Node ID documented | ❌ if missing — cannot verify later |
| Dimensions (width, height) per breakpoint tier | ❌ if component is responsive and tiers are missing |
| Padding (all applicable sides) with px values | ❌ if component has visible padding and values are missing |
| Typography per text element (size, weight, line-height, color) | ❌ if component has text and any property is missing |
| Background and border colors as hex values | ❌ if component has color and hex is missing |
| Border radius | ❌ if component has rounded corners and value is missing |
| Interactive state node IDs (hover, focus, disabled) | ❌ if component is interactive and state nodes are undocumented |
| Spacing between child elements (gap values) | ❌ if component has multiple children and gap is missing |

**Output:** Append to the design analysis file:

```markdown
## Completeness Audit

| Component | Node ID | Dimensions | Padding | Typography | Colors | Radius | States | Gaps | Status |
|---|---|---|---|---|---|---|---|---|---|
| [Name] | ✅/❌ | ✅/❌/N/A | ✅/❌/N/A | ✅/❌/N/A | ✅/❌/N/A | ✅/❌/N/A | ✅/❌/N/A | ✅/❌/N/A | PASS/FAIL |
```

If any component has FAIL status, resolve via Figma MCP or flag to the developer before presenting to Gate 1."

