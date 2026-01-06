---
name: linear-ticket-creator
description: Creates Linear tickets from a markdown file containing ticket definitions and dependencies. Reads a structured ticket file and creates issues in Linear with proper blocking relationships. Use when you have a ticket breakdown document ready to be imported into Linear.
allowed-tools: Bash, Read, Grep, Glob, AskUserQuestion
---

# Linear Ticket Creator

This skill reads a markdown file containing ticket definitions and their dependencies, then creates the tickets in Linear with proper blocking relationships.

## Goals

- Parse structured markdown files with ticket definitions
- Create tickets in Linear for a specified project/team
- Establish blocking relationships between tickets
- Never delete anything - only create
- Report all created tickets with links

## Prerequisites

The Linearis CLI tool must be installed and configured:

```bash
npm install -g --install-links czottmann/linearis
```

Authenticate with your Linear API token:
- Save token to `~/.linear_api_token` file

Get your API token from Linear: Settings > Security & Access > Personal API keys

## Expected Input Format

The skill expects a markdown file with tickets in this format:

```markdown
## Tickets

### T1: Ticket Title Here

**Labels:** backend, lab-agent
**Description:** (or just text under the heading)

Description text here...

#### Acceptance Criteria

- [ ] Criteria 1
- [ ] Criteria 2

#### Dependencies

- T2 (or "None")
```

The skill recognizes:
- Ticket IDs: `T1`, `T2`, `#1`, `#2`, etc.
- Dependencies: Listed under `#### Dependencies` or `**Dependencies:**`
- Labels: Under `**Labels:**` (comma-separated)
- Descriptions: Text content under the ticket heading

## Workflow

```
1. User provides path to ticket markdown file
    ↓
2. Parse file to extract tickets and dependencies
    ↓
3. Ask user for Linear project/team details
    ↓
4. Confirm tickets to be created
    ↓
5. Create tickets in Linear (in dependency order)
    ↓
6. Create blocking relations between tickets
    ↓
7. Report created tickets with links
```

## Instructions

### 1. Read and Parse the Ticket File

Read the provided markdown file and extract:

- **Ticket ID**: The identifier (T1, T2, #1, etc.)
- **Title**: The heading text after the ID
- **Description**: All text content under the ticket heading
- **Labels**: From `**Labels:**` line (comma-separated)
- **Dependencies**: From `#### Dependencies` section

Build a data structure like:

```
tickets = [
  {
    id: "T1",
    title: "Add Cloud Credentials Protobuf Enums",
    description: "Full description...",
    labels: ["backend", "lab-agent"],
    dependencies: []  // T1 has no dependencies
  },
  {
    id: "T2",
    title: "Add Cloud Credentials Protobuf Messages",
    description: "Full description...",
    labels: ["backend", "lab-agent"],
    dependencies: ["T1"]  // T2 depends on T1
  }
]
```

### 2. Ask User for Linear Configuration

Use AskUserQuestion to get:

1. **Team**: Which Linear team to create tickets in (e.g., "RAM", "Ramen")
2. **Project**: Which project to add tickets to (optional)
3. **Project Milestone**: Which milestone to associate (optional)
4. **Parent Ticket**: Parent issue if these are sub-tasks (optional)

Example questions:
```
Which Linear team should these tickets be created in?
- RAM
- Other (specify)

Which project should these tickets be added to?
- Instruqt 2.0 (NXT): Cloud Accounts support in Labs
- None
- Other (specify)
```

### 3. Confirm Before Creating

Show the user a summary of what will be created:

```markdown
## Tickets to Create

| # | Title | Labels | Dependencies |
|---|-------|--------|--------------|
| T1 | Add Protobuf Enums | backend, lab-agent | None |
| T2 | Add Protobuf Messages | backend, lab-agent | T1 |
| T3 | Add RPC Definition | backend, lab-agent | T2 |

**Team:** RAM
**Project:** Cloud Accounts support in Labs
**Total tickets:** 3
**Relations to create:** 2 blocking relationships

Proceed with creation?
```

### 4. Create Tickets in Dependency Order

Create tickets using topological sort order (dependencies first):

```bash
# Create a ticket
linearis issues create "Add Cloud Credentials Protobuf Enums" \
  --team "RAM" \
  --project "Instruqt 2.0 (NXT): Cloud Accounts support in Labs" \
  --labels "backend,lab-agent" \
  --description "$(cat <<'EOF'
## Description

Add the CloudProvider enum and update TabType enum...

## Acceptance Criteria

- [ ] Add CloudProvider enum
- [ ] Add TAB_TYPE_CLOUD_CREDENTIALS
- [ ] Run protobuf generation
EOF
)"
```

**Important:** Capture the created ticket identifier from the output. Linearis returns JSON with the created issue details.

```bash
# Parse the created ticket ID
TICKET_ID=$(linearis issues create "Title" --team RAM --description "Desc" | jq -r '.identifier')
echo "Created: $TICKET_ID"
```

### 5. Track Created Tickets

Maintain a mapping from local IDs to Linear identifiers:

```
T1 → RAM-301
T2 → RAM-302
T3 → RAM-303
```

### 6. Create Blocking Relations

After all tickets are created, establish blocking relationships using GraphQL:

```bash
# Get UUIDs for the tickets
T1_UUID=$(linearis issues read RAM-301 | jq -r '.id')
T2_UUID=$(linearis issues read RAM-302 | jq -r '.id')

# T1 blocks T2 (T2 depends on T1)
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw "{
    \"query\": \"mutation { issueRelationCreate(input: { issueId: \\\"$T1_UUID\\\", relatedIssueId: \\\"$T2_UUID\\\", type: blocks }) { success } }\"
  }"
```

**Relation semantics:**
- If T2 depends on T1, then T1 **blocks** T2
- `issueId` = the blocker (T1)
- `relatedIssueId` = the blocked issue (T2)

### 7. Report Results

Output a summary of what was created:

```markdown
## Created Tickets

| Local ID | Linear ID | Title | Link |
|----------|-----------|-------|------|
| T1 | RAM-301 | Add Protobuf Enums | [RAM-301](https://linear.app/instruqt/issue/RAM-301) |
| T2 | RAM-302 | Add Protobuf Messages | [RAM-302](https://linear.app/instruqt/issue/RAM-302) |
| T3 | RAM-303 | Add RPC Definition | [RAM-303](https://linear.app/instruqt/issue/RAM-303) |

## Created Relations

| Blocker | Blocks |
|---------|--------|
| RAM-301 | RAM-302 |
| RAM-302 | RAM-303 |

All 3 tickets created successfully with 2 blocking relationships.
```

## Parsing Guidelines

### Extracting Ticket Sections

Look for these patterns:

```markdown
### T1: Title
### #1: Title
### Ticket 1: Title
## T1: Title
```

### Extracting Dependencies

Look for:

```markdown
#### Dependencies
- T1
- T2

**Dependencies:** T1, T2

Dependencies: T1
```

Parse dependency references:
- `T1`, `T2`, `#1`, `#2`
- `None` or empty means no dependencies

### Extracting Labels

```markdown
**Labels:** backend, lab-agent
Labels: frontend, backend
**Type:** Task **Labels:** backend
```

### Extracting Description

Everything between the ticket heading and the next ticket heading (or section), excluding metadata lines.

## Error Handling

### Ticket Creation Fails

If a ticket fails to create:
1. Log the error
2. Continue with other tickets that don't depend on it
3. Report which tickets were skipped
4. Do NOT attempt to delete any created tickets

### Relation Creation Fails

If a relation fails:
1. Log the error
2. Continue with other relations
3. Report which relations failed
4. Suggest manual fix

### Duplicate Detection

Before creating:
1. Search for existing tickets with similar titles
2. Warn user if potential duplicates found
3. Ask whether to proceed

```bash
linearis issues search "Add Protobuf Enums" --team RAM
```

## Safety Rules

1. **Never delete tickets** - Only create
2. **Never modify existing tickets** - Only create new ones
3. **Confirm before creating** - Always show summary first
4. **Report everything** - Log all actions taken
5. **Handle partial success** - Continue on errors, report at end

## Quick Reference

### Linearis Commands

```bash
# Create ticket
linearis issues create "Title" \
  --team "RAM" \
  --project "Project Name" \
  --labels "label1,label2" \
  --description "Description" \
  --parent-ticket RAM-100 \
  --project-milestone "Milestone Name"

# Read ticket (get UUID)
linearis issues read RAM-123 | jq -r '.id'

# Search for potential duplicates
linearis issues search "title query" --team RAM
```

### GraphQL for Relations

```bash
# Create blocking relation (A blocks B)
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{
    "query": "mutation { issueRelationCreate(input: { issueId: \"<A-UUID>\", relatedIssueId: \"<B-UUID>\", type: blocks }) { success } }"
  }'
```

## When to Use This Skill

Use `linear-ticket-creator` when:
- You have a markdown file with ticket definitions
- You want to bulk-create tickets from a technical design
- You need to establish dependency relationships
- You're importing work items from a planning document

**Do NOT use** for:
- Updating existing tickets (use `linear-updater`)
- Breaking down a single ticket (use `ticket-breakdown`)
- Investigating ticket requirements (use `linear-investigator`)
