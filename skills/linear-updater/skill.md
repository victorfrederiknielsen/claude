---
name: linear-updater
description: Updates Linear tickets based on new information. Can modify title, description, state, labels, priority, parent tickets, and issue relations (blocked/blocks). Use when tickets need to be updated based on technical designs, implementation changes, or re-sequencing work.
allowed-tools: Bash, Read, Grep, Glob, WebFetch, AskUserQuestion
---

# Linear Ticket Updater

This skill updates Linear tickets based on new information such as technical designs, implementation changes, or re-sequencing of work. It can modify ticket properties and manage issue relations (blocked by/blocks).

## Prerequisites

The Linearis CLI tool must be installed and configured:

```bash
npm install -g --install-links czottmann/linearis
```

Authenticate with your Linear API token:
- Save token to `~/.linear_api_token` file

Get your API token from Linear: Settings > Security & Access > Personal API keys

## Capabilities

### Via Linearis CLI

| Property | Command Flag | Example |
|----------|--------------|---------|
| Title | `-t, --title` | `linearis issues update RAM-123 -t "New title"` |
| Description | `-d, --description` | `linearis issues update RAM-123 -d "New desc"` |
| State | `-s, --state` | `linearis issues update RAM-123 -s "In Progress"` |
| Priority | `-p, --priority` | `linearis issues update RAM-123 -p 2` |
| Labels | `--labels` | `linearis issues update RAM-123 --labels "frontend,backend"` |
| Parent ticket | `--parent-ticket` | `linearis issues update RAM-123 --parent-ticket RAM-100` |
| Clear parent | `--clear-parent-ticket` | `linearis issues update RAM-123 --clear-parent-ticket` |
| Project milestone | `--project-milestone` | `linearis issues update RAM-123 --project-milestone "MVP"` |
| Cycle | `--cycle` | `linearis issues update RAM-123 --cycle "Sprint 5"` |

### Via GraphQL API (Relations)

| Relation Type | Description |
|---------------|-------------|
| `blocks` | This issue blocks another issue |
| `duplicate` | This issue is a duplicate of another |
| `related` | Issues are generally related |
| `similar` | Issues are similar |

## Workflow

```
1. Analyze new information (design doc, requirements, etc.)
    ↓
2. Fetch current ticket state
    ↓
3. Determine what needs updating
    ↓
4. Confirm changes with user (if significant)
    ↓
5. Apply updates via linearis CLI and/or GraphQL API
    ↓
6. Report changes made
```

## Instructions

### 1. Fetch Current Ticket State

Always read the current ticket before making changes:

```bash
linearis issues read RAM-123
```

To get relations, use GraphQL:

```bash
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{
    "query": "{ issue(id: \"RAM-123\") { id identifier title relations { nodes { id type relatedIssue { identifier title } } } inverseRelations { nodes { id type issue { identifier title } } } } }"
  }'
```

### 2. Update Basic Properties

Use linearis CLI for simple property updates:

```bash
# Update title
linearis issues update RAM-123 -t "Updated: Add cloud credentials support"

# Update description
linearis issues update RAM-123 -d "## Overview\n\nNew description here..."

# Update state
linearis issues update RAM-123 -s "In Progress"

# Update priority (1=Urgent, 2=High, 3=Medium, 4=Low)
linearis issues update RAM-123 -p 2

# Update labels (comma-separated)
linearis issues update RAM-123 --labels "frontend,backend"

# Set parent ticket
linearis issues update RAM-123 --parent-ticket RAM-100
```

### 3. Manage Issue Relations (GraphQL)

Relations require the GraphQL API. First, get the issue UUIDs:

```bash
# Get UUID for an issue
linearis issues read RAM-123 | jq -r '.id'
```

#### Create a Relation

```bash
# Make RAM-242 block RAM-247 (RAM-242 must be done before RAM-247)
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{
    "query": "mutation { issueRelationCreate(input: { issueId: \"<UUID-of-RAM-242>\", relatedIssueId: \"<UUID-of-RAM-247>\", type: blocks }) { success issueRelation { id type } } }"
  }'
```

**Relation direction:**
- `issueId` **blocks** `relatedIssueId`
- So if A must be done before B: A.blocks(B) means issueId=A, relatedIssueId=B

#### Delete a Relation

```bash
# First get the relation ID from the query
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{
    "query": "mutation { issueRelationDelete(id: \"<relation-uuid>\") { success } }"
  }'
```

#### Update a Relation Type

```bash
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{
    "query": "mutation { issueRelationUpdate(id: \"<relation-uuid>\", input: { type: related }) { success } }"
  }'
```

## Usage Scenarios

### Scenario 1: Update Tickets from Technical Design

When a technical design document changes the scope or sequence of work:

1. Read the design document
2. Fetch all affected tickets
3. Compare current state with design requirements
4. Propose changes to user
5. Apply approved updates

### Scenario 2: Re-sequence Blocked/Blocking Relations

When implementation order needs to change:

1. Query current relations for all tickets
2. Identify incorrect blocking relationships
3. Delete outdated relations
4. Create new relations with correct sequence
5. Report the updated dependency graph

### Scenario 3: Bulk Description Update

When acceptance criteria or implementation details change:

1. Identify tickets needing updates
2. Read current descriptions
3. Generate updated descriptions
4. Confirm with user
5. Apply updates via linearis

## Guidelines

### Before Making Changes

1. **Always fetch current state first** - Don't assume ticket content
2. **Confirm significant changes** - Use AskUserQuestion for title changes, relation changes, or state transitions
3. **Preserve existing content** - When updating descriptions, incorporate existing content unless explicitly replacing

### Change Confirmation

For these changes, always confirm with the user first:
- Title changes
- State transitions
- Adding/removing blocking relations
- Changing parent tickets

For these changes, proceed without confirmation:
- Minor description updates based on explicit user request
- Adding labels mentioned by user
- Priority changes requested by user

### Reporting Changes

After making updates, report:
- What was changed
- Previous vs new values
- Any relations added/removed
- Link to the updated ticket(s)

## Quick Reference

### Linearis Commands

```bash
# Read
linearis issues read RAM-123
linearis issues search "query" --team RAM

# Update properties
linearis issues update RAM-123 -t "Title" -d "Desc" -s "State" -p 2
linearis issues update RAM-123 --labels "a,b" --parent-ticket RAM-100
linearis issues update RAM-123 --project-milestone "Milestone" --cycle "Sprint"
```

### GraphQL Queries

```bash
# Query with relations
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{"query": "{ issue(id: \"RAM-123\") { id identifier title state { name } relations { nodes { type relatedIssue { identifier } } } inverseRelations { nodes { type issue { identifier } } } } }"}'
```

### IssueRelationType Values

| Value | Meaning |
|-------|---------|
| `blocks` | This issue blocks another |
| `duplicate` | Duplicate of another issue |
| `related` | Generally related |
| `similar` | Similar issue |

## When to Use This Skill

Use `linear-updater` when:
- Technical design changes require ticket updates
- Implementation order/sequencing needs adjustment
- Ticket descriptions need updating based on new information
- Blocking relationships need to be added or modified
- Multiple tickets need coordinated updates

**Do NOT use** for:
- Creating new tickets (use linearis directly)
- Just reading tickets (use `linear-planner` or `linear-investigator`)
- Breaking down tickets (use `ticket-breakdown`)
