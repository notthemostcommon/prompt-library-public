---
description: Generate AEM Content Fragment Model implementation guide from story requirements
globs: *
alwaysApply: false
---
"do fas; Generate an AEM Content Fragment Model implementation guide for [STORY_ID].

INTENTION: Transform story requirements into actionable AEM authoring instructions. Analyze existing models for reuse, provide copy-pastable field specifications, and include only essential information.

CONTEXT FILES TO REFERENCE:
- docs/aem/content-fragment-models.md (existing models)
- docs/aem/aem-authoring-quick-reference.md (naming conventions and path patterns)
- lib/aem/constants.ts (actual paths in use)

PROCESS:

1. Read specifications/[STORY_ID]/story.md and extract content structure requirements
2. Review docs/aem/content-fragment-models.md to identify reusable models
3. For each field, determine: existing model reference OR new field
4. Apply reusability best practice: Fragment References for shared content (images, links, rich text)
5. If ambiguous whether to reuse, ask user interactively
6. Use path pattern: /content/dam/[tenant]/en/web/[project]/{type-plural}/ (replace [tenant] and [project] with your AEM configuration)
7. Ensure accessibility: images need altText, interactive elements need labels

OUTPUT FORMAT:

Create specifications/[STORY_ID]/[STORY_ID]-AEM-CFM-GUIDE.md with these sections:

## Executive Summary
Brief overview with counts: new models, reused models, fragments needed, estimated time

## 1. Model Reuse Analysis
Tables only:
- Existing models to reuse (model name, purpose, rationale)
- New models to create (model name, purpose, rationale)
- Include any design decisions that need confirmation

## 2. Folder Structure
Paths using /content/dam/[tenant]/en/web/[project]/ pattern with brief rationale

## 3. Model Creation Instructions
For each new model:

### Model Name: [ModelName]
**Path**: /conf/[tenant]/settings/dam/cfm/models/[kebab-case]
**Description**: [Single sentence - copy-pastable to AEM]

**Fields**:
| Field Label | Property Name | Type | Required | Validation (Regex) | Validation Notes | Description (for AEM) |
|-------------|---------------|------|----------|-------------------|-----------------|----------------------|
| [Label] | [camelCase] | [AEM type] | Yes/No | [regex pattern only] | [What regex does] | [Copy-paste to AEM description field] |

Notes:
- Validation (Regex) column: copy-paste directly into AEM validation field - no explanatory text
- Validation Notes: human-readable explanation of validation rule
- Leave Validation columns empty if no validation needed

**Field Details** (only if type needs explanation):
- [fieldName]: Why [Type] - brief rationale

Keep model instructions concise - one table per model, field details only when non-obvious

## 4. Content Fragment Creation
Ordered steps (dependencies first). Use tables for multiple similar fragments:

| Fragment Name | Field 1 | Field 2 | Field 3 |
|---------------|---------|---------|---------|
| [name] | [value] | [value] | [value] |

Include only: fragment names, field values, locations. Skip step-by-step AEM UI navigation.

## 5. GraphQL Query Reference
Query the frontend will use with expected response structure (abbreviated with comments)

## 6. Testing Checklist
Single consolidated checklist covering: AEM validation, GraphQL, frontend integration, accessibility, responsive
Limit to 10-15 most critical items

## 7. Common Issues (Top 3 Only)
Most likely problems with quick solutions

## 8. Acceptance Criteria Validation
Map story acceptance criteria to implementation with checkboxes

CONCISENESS GUIDELINES:
- Use tables wherever possible instead of prose
- Field specifications: essential info only (Label, Property, Type, Required, Validation, Description)
- Descriptions must be copy-pastable directly into AEM
- Skip AEM UI navigation steps (authors know how to create fragments)
- Limit common issues to 3 most critical
- Testing checklist: 10-15 items max
- No verbose explanations - bullet points and tables preferred
- Omit sections 9-10 (Maintenance Notes, Next Steps) unless critical to story"

