---
name: linear-reader
description: Reads Linear items including tickets/issues, projects, documents, cycles, milestones, and more. Parses Linear URLs to automatically detect and fetch the appropriate resource type. Use when you need to read or understand any Linear item.
allowed-tools: Bash, Read, WebFetch
---

# Linear Reader

This skill reads various Linear resources and presents them in a structured format. It can handle issues, projects, documents, cycles, milestones, and more by parsing Linear URLs or using identifiers directly.

## Goals

- **Universal reader**: Read any Linear resource type from a URL or identifier
- **Auto-detect type**: Parse Linear URLs to determine the resource type
- **Structured output**: Present information in a consistent, readable format
- **Fetch related data**: Include relevant context like comments, attachments, and relations

## Prerequisites

The Linearis CLI tool must be installed and configured:

```bash
npm install -g --install-links czottmann/linearis
```

Authenticate with your Linear API token:
- Set `LINEAR_API_TOKEN` environment variable, OR
- Save token to `~/.linear_api_token` file

Get your API token from Linear: Settings > Security & Access > Personal API keys

## Supported Resource Types

| Resource | URL Pattern | CLI Support |
|----------|-------------|-------------|
| Issue/Ticket | `/issue/{IDENTIFIER}` | `linearis issues read` |
| Project | `/project/{ID}` | GraphQL API |
| Document | `/document/{ID}` | GraphQL API |
| Cycle/Sprint | `/cycle/{ID}` | `linearis cycles read` |
| Milestone | `/project/{ID}/milestone/{ID}` | `linearis project-milestones read` |

## URL Patterns

Linear uses these URL formats:

```
https://linear.app/{workspace}/issue/{IDENTIFIER}/...
https://linear.app/{workspace}/project/{PROJECT-ID}/...
https://linear.app/{workspace}/document/{DOC-ID}/...
https://linear.app/{workspace}/cycle/{CYCLE-ID}/...
https://linear.app/{workspace}/team/{TEAM}/cycle/{CYCLE-ID}/...
```

## Instructions

### 1. Parse the Linear URL or Identifier

Determine the resource type from the URL:

```
URL contains "/issue/" → Issue
URL contains "/project/" and "/milestone/" → Milestone
URL contains "/project/" → Project
URL contains "/document/" → Document
URL contains "/cycle/" → Cycle
Plain identifier like "RAA-123" → Issue
```

### 2. Read the Resource

#### Reading Issues

Use the linearis CLI:

```bash
linearis issues read {IDENTIFIER}
```

Returns JSON with:
- `id`: UUID
- `identifier`: Ticket ID (e.g., RAA-123)
- `title`: Issue title
- `description`: Full description (markdown)
- `state`: Current state object
- `priority`: Priority level (0-4)
- `labels`: Array of labels
- `team`: Team information
- `assignee`: Assigned user
- `parentIssue`: Parent ticket (if subtask)
- `subIssues`: Child issues
- `project`: Associated project
- `cycle`: Current cycle/sprint
- `embeds`: Attached files with URLs
- `comments`: Array of comments
- `relations`: Blocking/related issues
- `createdAt`, `updatedAt`: Timestamps

**Output format for issues:**

```markdown
# {IDENTIFIER}: {title}

**Status:** {state.name}
**Priority:** {priority} ({priorityLabel})
**Assignee:** {assignee.name or "Unassigned"}
**Team:** {team.name}
**Project:** {project.name or "None"}
**Cycle:** {cycle.name or "None"}
**Labels:** {labels, comma-separated}
**Created:** {createdAt}
**Updated:** {updatedAt}

## Description

{description}

## Attachments

{list embeds with download URLs}

## Comments

{list comments with author and content}

## Relations

- **Blocked by:** {issues blocking this one}
- **Blocks:** {issues this one blocks}
- **Related:** {related issues}

## Sub-issues

{list sub-issues if any}

---
Raw JSON available via: linearis issues read {IDENTIFIER}
```

#### Reading Projects

Use GraphQL API:

```bash
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{
    "query": "{ project(id: \"{PROJECT-ID}\") { id name description state startDate targetDate progress { ... } lead { name } members { nodes { name } } teams { nodes { name } } issues { nodes { identifier title state { name } } } milestones { nodes { id name targetDate } } } }"
  }'
```

**Output format for projects:**

```markdown
# Project: {name}

**State:** {state}
**Lead:** {lead.name}
**Progress:** {progress}%
**Start Date:** {startDate}
**Target Date:** {targetDate}
**Teams:** {teams, comma-separated}

## Description

{description}

## Members

{list members}

## Milestones

| Milestone | Target Date |
|-----------|-------------|
{milestone rows}

## Issues ({count})

| ID | Title | Status |
|----|-------|--------|
{issue rows}

---
```

#### Reading Documents

Use GraphQL API:

```bash
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{
    "query": "{ document(id: \"{DOC-ID}\") { id title content slugId creator { name } project { name } createdAt updatedAt } }"
  }'
```

**Output format for documents:**

```markdown
# Document: {title}

**Creator:** {creator.name}
**Project:** {project.name or "None"}
**Created:** {createdAt}
**Updated:** {updatedAt}

---

{content}

---
```

#### Reading Cycles (Sprints)

Use linearis CLI:

```bash
linearis cycles read "{CYCLE-NAME}" --team {TEAM}
# or by UUID
linearis cycles read {CYCLE-UUID}
```

Returns cycle details including all issues in the cycle.

**Output format for cycles:**

```markdown
# Cycle: {name}

**Team:** {team.name}
**Start Date:** {startsAt}
**End Date:** {endsAt}
**Progress:** {progress}%
**Status:** {Active/Completed/Upcoming}

## Issues ({count})

### In Progress
| ID | Title | Assignee |
|----|-------|----------|
{in-progress issues}

### Todo
{todo issues}

### Done
{completed issues}

---
```

#### Reading Milestones

Use linearis CLI:

```bash
linearis project-milestones read "{MILESTONE-NAME}" --project "{PROJECT-NAME}"
# or by UUID
linearis project-milestones read {MILESTONE-UUID}
```

**Output format for milestones:**

```markdown
# Milestone: {name}

**Project:** {project.name}
**Target Date:** {targetDate}
**Status:** {Planned/In Progress/Completed}

## Description

{description}

## Issues ({count})

| ID | Title | Status | Assignee |
|----|-------|--------|----------|
{issue rows}

---
```

### 3. Handle Embedded Files

If the resource has attachments (embeds), list them:

```bash
# Download if needed
linearis embeds download "{URL}" -o /tmp/filename
```

### 4. Fetch Additional Context

For issues, optionally fetch:

**Relations via GraphQL:**

```bash
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{
    "query": "{ issue(id: \"{IDENTIFIER}\") { relations { nodes { type relatedIssue { identifier title } } } inverseRelations { nodes { type issue { identifier title } } } } }"
  }'
```

## Quick Reference

### Linearis CLI Commands

```bash
# Issues
linearis issues read {IDENTIFIER}
linearis issues list --team {TEAM}
linearis issues search "query" --team {TEAM}

# Cycles
linearis cycles list --team {TEAM}
linearis cycles read "{NAME}" --team {TEAM}

# Milestones
linearis project-milestones list --project "{PROJECT}"
linearis project-milestones read "{NAME}" --project "{PROJECT}"

# Projects
linearis projects list

# Labels
linearis labels list

# Embeds
linearis embeds download "{URL}" -o {OUTPUT}
```

### GraphQL Queries

```bash
# Read document
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{
    "query": "{ document(id: \"{ID}\") { id title content creator { name } createdAt updatedAt } }"
  }'

# Read project with details
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{
    "query": "{ project(id: \"{ID}\") { id name description state lead { name } issues { nodes { identifier title state { name } } } } }"
  }'

# Search documents
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{
    "query": "{ documentSearch(term: \"search term\", first: 10) { nodes { id title project { name } } } }"
  }'

# Read issue with relations
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{
    "query": "{ issue(id: \"{IDENTIFIER}\") { id identifier title description state { name } priority labels { nodes { name } } assignee { name } relations { nodes { type relatedIssue { identifier title } } } inverseRelations { nodes { type issue { identifier title } } } comments { nodes { body user { name } createdAt } } } }"
  }'
```

## Usage Examples

### Example 1: Read from URL

User provides: `https://linear.app/instruqt/issue/RAA-456/add-widget-filtering`

1. Parse URL → type: issue, identifier: RAA-456
2. Run: `linearis issues read RAA-456`
3. Format and present the issue details

### Example 2: Read Project

User provides: `https://linear.app/instruqt/project/cloud-accounts-12345`

1. Parse URL → type: project, id: cloud-accounts-12345
2. Query GraphQL API for project details
3. Format and present project with issues and milestones

### Example 3: Read Document

User provides: `https://linear.app/instruqt/document/technical-spec-abc123`

1. Parse URL → type: document, id: technical-spec-abc123
2. Query GraphQL API for document content
3. Present the full document content

### Example 4: Read by Identifier

User provides: `RAA-789`

1. Recognize as issue identifier
2. Run: `linearis issues read RAA-789`
3. Format and present the issue

## Error Handling

### Resource Not Found

If the resource doesn't exist:
```
Error: Could not find {type} with identifier {id}
Please verify the URL or identifier is correct.
```

### Authentication Issues

If authentication fails:
```
Error: Authentication failed. Please verify your Linear API token.
- Check ~/.linear_api_token exists and contains a valid token
- Or set LINEAR_API_TOKEN environment variable
- Get a token from: Linear > Settings > Security & Access > Personal API keys
```

### Invalid URL Format

If the URL can't be parsed:
```
Error: Could not parse Linear URL. Expected format:
- https://linear.app/{workspace}/issue/{IDENTIFIER}
- https://linear.app/{workspace}/project/{ID}
- https://linear.app/{workspace}/document/{ID}
- https://linear.app/{workspace}/cycle/{ID}

Or provide an identifier directly (e.g., RAA-123)
```

## When to Use This Skill

Use `linear-reader` when:
- You need to read any Linear resource (issue, project, document, cycle, milestone)
- You have a Linear URL and want to fetch its contents
- You want to understand the details of a Linear item before working on it
- You need to fetch document content from Linear
- You want to see the issues in a project or cycle

**This skill is read-only** - it does not modify any Linear resources.

For other operations:
- Creating/updating tickets: use `linear-updater` or `linear-ticket-creator`
- Planning implementation: use `linear-planner`
- Investigating requirements: use `linear-investigator`
- Breaking down tickets: use `ticket-breakdown`
