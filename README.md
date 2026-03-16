# Prompt Library

A curated collection of development rules and prompt templates for AI-assisted software development. Use these structured prompts to maintain consistency and quality when working with AI coding assistants.

## Quick Start

Add this library to your project as a git submodule:

```bash
# Add the submodule to your project
git submodule add git@github.com:your-org/prompt-library.git .cursor

# Initialize and update the submodule
git submodule update --init --recursive

# In the future, update to latest rules
git submodule update --remote .cursor

```

## What's Included

### Rules (`.cursor/rules`)
Development standards and architectural patterns that AI assistants should follow:

- **core-dev-rules.mdc** - Recommended rules from Cursor.
- **csharp-dotnet-coding-standards.mdc** - Structured rules around C#

### Skills (`.cursor/skills`)
On-demand workflow guidance that the agent loads when performing specific tasks:

- **commit-messages** - Conventional commit message format with story/ticket references
- **bdd-story-analysis** - BDD story analysis and test planning
- **edge-case-heuristics** - Edge case testing validation checklists
- **test-review** - Test suite coverage and completeness review
- **test-usefulness** - Test quality and usefulness evaluation
- **troubleshooting** - Debugging protocol and error investigation

Rules provide structured guidance with:
- Core principles and context
- Tiered requirements (Must/Should/Could Have)
- Code examples and anti-patterns
- Decision frameworks for edge cases

### Commands (`.cursor/commands`)
TODO: Insert the explanation of these commands

## Usage

It is best practice to reference rules in your prompts. Cursor in most cases will include these
automatically; however, we've seen cases where this does not happen.

## Benefits

- **Consistency** - Standardized patterns across projects and team members
- **Quality** - Battle-tested rules based on industry best practices  
- **Efficiency** - Reusable templates reduce prompt engineering overhead
- **Maintainability** - Centralized rules that evolve with your practices

### MCP Commands
- `analyze-design-requirements`: This command leverages the mcp server for figma to interrogate the designs
- `fetch-jira-story`: This command leverages the jira mcp server to install the mcp server.


### MCP Server Configuration:
```{
  "mcpServers": {
    "Figma": {
      "url": "https://mcp.figma.com/mcp",
      "headers": {}
    },
    "Atlassian-MCP-Server": {
      "url": "https://mcp.atlassian.com/v1/sse"
    }
  }
}