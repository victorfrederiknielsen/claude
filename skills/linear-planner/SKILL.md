---
name: linear-planner
description: Takes a Linear ticket URL and creates a detailed implementation plan. Use when the user provides a Linear ticket link and wants to understand how to solve it. Fetches ticket details and generates a structured plan.
allowed-tools: Bash, Read, Grep, Glob, WebFetch, AskUserQuestion
---

# Linear Ticket Planner

This skill analyzes Linear tickets and creates comprehensive implementation plans. It fetches ticket details using the Linearis CLI tool and generates a structured plan based on the ticket type and requirements.

## Prerequisites

The Linearis CLI tool must be installed and configured:

```bash
npm install -g --install-links czottmann/linearis
```

You must authenticate with your Linear API token:
- Set `LINEAR_API_TOKEN` environment variable, OR
- Save token to `~/.linear_api_token` file, OR
- Use `--api-token` flag with each command

Get your API token from Linear: Settings → Security & Access → Personal API keys

## Usage

When the user provides a Linear ticket URL (e.g., `https://linear.app/instruqt/issue/RAA-123/...`), this skill will:

1. Extract the ticket identifier (e.g., `RAA-123`)
2. Fetch full ticket details using `linearis issues read`
3. Analyze the ticket content, description, labels, and related issues
4. Generate a structured implementation plan

## Instructions

### 1. Extract Ticket Identifier

From the Linear URL, extract the ticket identifier:
- URL format: `https://linear.app/{workspace}/issue/{IDENTIFIER}/...`
- Example: `https://linear.app/instruqt/issue/RAA-993/...` → `RAA-993`

### 2. Fetch Ticket Details

Use the Linearis CLI to fetch comprehensive ticket information:

```bash
linearis issues read {IDENTIFIER}
```

This returns JSON with:
- `identifier`: Ticket ID
- `title`: Ticket title
- `description`: Full description (markdown)
- `state`: Current state (Todo, In Progress, Done, etc.)
- `priority`: Priority level (0-4, where 0 is None, 1 is Urgent)
- `labels`: Array of label objects with names
- `team`: Team information
- `assignee`: Assigned user (if any)
- `parentIssue`: Parent ticket (if this is a subtask)
- `subIssues`: Array of subtasks
- `project`: Associated project
- `cycle`: Sprint/cycle information
- `embeds`: Attached files with download URLs

### 3. Analyze Ticket Context

Examine the ticket to understand:
- **Type of work**: Is it a bug fix, feature, refactor, or investigation?
- **Scope**: Is it a small fix or large feature? Does it have subtasks?
- **Dependencies**: Are there parent issues or related tickets mentioned?
- **Requirements**: What does the description specify?
- **Acceptance criteria**: What needs to be done for completion?
- **Technology stack**: Is this primarily React frontend, Go backend, or both?

### 4. Determine Planning Strategy

Based on the ticket analysis, determine which specialized planner to use:

- **React work**: Use the `react-planner` skill for:
  - Frontend components and UI work
  - React state management
  - Client-side logic and interactions
  - Any work primarily in src/ with .tsx/.jsx files

- **Go work**: Use the `go-planner` skill for:
  - Backend services and APIs
  - Database operations
  - Server-side business logic
  - Any work primarily in Go packages

- **Full-stack work**: Use BOTH skills sequentially:
  1. Use `react-planner` for the frontend portion
  2. Use `go-planner` for the backend portion
  3. Combine both plans with integration notes

- **Other work**: If the ticket is for infrastructure, DevOps, or other non-React/Go work, continue with general planning below.

### 5. Search Codebase for Context (for non-specialized planning only)

If not delegating to specialized planners, identify relevant parts of the codebase:
- Use `Grep` to search for related code, components, or functions
- Use `Glob` to find relevant files
- Use `Read` to examine existing implementations
- Look for patterns, similar features, or affected areas

### 6. Generate Implementation Plan (for non-specialized planning only)

Create a structured plan that includes:

**Overview**
- Ticket summary and current state
- Type of work (bug/feature/refactor)
- Estimated complexity

**Analysis**
- Key requirements from the description
- Affected areas of the codebase
- Related files and components found

**Implementation Steps**
- Numbered list of concrete, actionable steps
- Each step should be specific and measurable
- Include file paths where changes are needed
- Note any dependencies between steps

**Technical Considerations**
- Potential challenges or edge cases
- Testing requirements
- Performance implications
- Breaking changes or migration needs

**Next Actions**
- Recommend which specialized skill to use for implementation
- Note any questions that need clarification
- Identify any missing information

### 7. Plan Format Example

```markdown
# Plan for {IDENTIFIER}: {Title}

## Overview
- **Status**: {state}
- **Priority**: {priority}
- **Type**: {Bug Fix/Feature/Refactor/Investigation}
- **Complexity**: {Low/Medium/High}

## Ticket Details
{Brief summary of the ticket description and key requirements}

## Codebase Analysis
{List of relevant files, components, and patterns found}

## Implementation Plan

1. {Step 1 with file paths}
2. {Step 2 with specific actions}
3. {Step 3 with testing requirements}
...

## Technical Considerations
- {Challenge or edge case 1}
- {Testing approach}
- {Performance notes}

## Recommended Next Steps
- **For React work**: Delegate to `react-planner` skill
- **For Go work**: Delegate to `go-planner` skill
- **For full-stack work**: Use both planners sequentially
- **For implementation**: After planning, use `react-executor` or `go-executor` skills
- Clarify: {any questions}
```

## Handling Different Ticket Types

### Bug Fixes
1. Reproduce the issue
2. Identify root cause
3. Plan the fix with minimal changes
4. Add regression tests

### New Features
1. Review requirements and acceptance criteria
2. Identify affected components
3. Plan incremental implementation
4. Consider testing strategy and documentation

### Refactoring
1. Understand current implementation
2. Identify code smells or issues
3. Plan refactoring steps that maintain functionality
4. Ensure test coverage before and after

### Investigations
1. Define what needs to be researched
2. Plan exploration approach
3. Identify what to document
4. Determine success criteria

## Delegation Strategy

When you identify React or Go work:

1. **Prepare context for delegation**: Summarize the ticket requirements clearly
2. **Invoke specialized planner**: Use the Skill tool to call `react-planner` or `go-planner`
3. **Pass ticket context**: Provide the ticket details and your initial analysis
4. **Let specialized planner work**: They will explore the codebase and create detailed plans
5. **Return the specialized plan**: Present the plan from the specialized planner to the user

Example delegation:
```
Based on the ticket analysis, this is React frontend work involving component changes.
I'll use the react-planner skill to create a detailed implementation plan.

[Invoke react-planner skill with ticket context]
```

## Notes

- Always fetch fresh ticket data to ensure accuracy
- Delegate to specialized planners (react-planner, go-planner) when appropriate
- If the ticket has subtasks, consider if each needs its own plan
- Check for linked PRs or commits that might provide context
- The plan should be detailed enough to hand off but flexible enough to adapt
- This skill only creates the plan - actual implementation uses executor skills
