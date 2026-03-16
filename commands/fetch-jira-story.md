---
description: Fetch Jira issue details for story analysis
globs: *
alwaysApply: false
---

Fetch the details of Jira issue [STORY_ID] (summary, status, assignee, story points, description, acceptance criteria, latest comments, and links).

INTENTION: Retrieve comprehensive story information from Jira to enable analysis, planning, and implementation work.

The fetch should include:
- Summary and description
- Status and priority
- Assignee and reporter
- Story points (if set)
- Acceptance criteria from description
- Latest comments
- Linked issues (blockers, dependencies, related stories)
- Technical considerations
- Out of scope items

Present the information in a clear, structured format that can be used for:
- Story decomposition
- Task planning
- Implementation context
- Dependency tracking

