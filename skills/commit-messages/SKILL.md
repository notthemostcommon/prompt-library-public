---
name: commit-messages
description: Conventional commit message format rules with story/ticket references
---

# Commit Message Rules

Follow the conventional commits format with story/ticket references:

## Format

```
<type>(<scope>): <description>
```

## Types

- `feat`: new feature
- `fix`: bug fix
- `docs`: documentation changes
- `style`: formatting, missing semicolons, etc.
- `refactor`: code change that neither fixes a bug nor adds a feature
- `test`: adding missing tests or correcting existing tests
- `chore`: changes to build process or auxiliary tools

## Scope

- Use the story/ticket number (e.g., `PROJ-24`, `PROJ-5`)
- For general changes without a specific ticket, use a descriptive scope (e.g., `config`, `scripts`, `infra`)

## Description

- Use lowercase
- Use imperative mood ("add" not "added" or "adds")
- No period at the end
- Be concise but descriptive

## Examples

```
feat(PROJ-24): add epic/story structure, execution plan and initial implementation
fix(PROJ-25): resolve authentication error in bootstrap script
docs(PROJ-5): update setup instructions for new deployment process
chore(config): update required APIs list for infrastructure
```
