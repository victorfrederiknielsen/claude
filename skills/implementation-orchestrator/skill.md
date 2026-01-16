# Implementation Orchestrator

Orchestrates the implementation of a sequence of Linear tickets, spawning specialized sub-agents for each piece of work. Handles the full lifecycle: planning, implementing, testing, PR creation, review, and ticket state management.

## Goals

- Execute tickets in proper dependency order
- Spawn specialized sub-agents (Go, React, etc.) for implementation
- Ensure TDD practices and CI passing
- Create draft PRs and self-review before human review
- Manage Linear ticket states throughout the process
- Parallelize where possible (exploration, validation)

## Prerequisites

- Linearis CLI: `npm install -g --install-links czottmann/linearis`
- Linear API token at `~/.linear_api_token`
- GitHub CLI (`gh`) authenticated
- Access to relevant repositories

## Workflow Overview

```
1. Read ticket sequence (from milestone, project, or list)
   ↓
2. Determine ready tickets (unblocked)
   ↓
3. For each ready ticket:
   a. Move to "In Progress"
   b. Navigate to correct repo
   c. Ensure on main/master, pull latest
   d. Create ticket branch
   e. Spawn implementation sub-agent
   f. Run tests, ensure CI passes
   g. Create draft PR
   h. Spawn review sub-agent
   i. Post review as PR comment
   j. Assess readiness → "In Review" or "Blocked"
   ↓
4. Repeat for next unblocked tickets
```

## Instructions

### Phase 1: Read Ticket Sequence

Get the ordered list of tickets to implement:

**From Project Milestone:**
```bash
linearis project-milestones read "{MILESTONE-NAME}" --project "{PROJECT}"
# or by UUID
linearis project-milestones read {MILESTONE-UUID}
```

**From List of Ticket IDs:**
```bash
# Read each ticket
for id in RAM-306 RAM-307 RAM-308 RAM-309 RAM-310; do
  linearis issues read $id
done
```

**Parse blocking relations:**
```bash
curl -s -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(cat ~/.linear_api_token)" \
  --data-raw '{
    "query": "{ issue(id: \"{IDENTIFIER}\") { relations { nodes { type relatedIssue { identifier title state { name } } } } inverseRelations { nodes { type issue { identifier title state { name } } } } } }"
  }'
```

Build dependency graph:
```
RAM-306 → (no deps, ready)
RAM-307 → blocked by RAM-306
RAM-308 → blocked by RAM-307
...
```

### Phase 2: Identify Ready Tickets

A ticket is ready when:
1. All blocking tickets are "Done"
2. Ticket is not already "In Progress", "In Review", or "Done"

```bash
# Check ticket state
linearis issues read {ID} | jq -r '.state.name'
```

### Phase 3: Execute Ticket Implementation

For each ready ticket:

#### 3.1: Move to "In Progress"

```bash
linearis issues update {IDENTIFIER} --state "In Progress"
```

#### 3.2: Determine Repository and Navigate

Parse the ticket to identify the target repository:
- Look for repo mentions in title/description
- Check labels (e.g., "frontend", "backend", "lab-agent")
- Use ticket metadata

Repository mapping example:
```
frontend → /Users/victornielsen/Documents/instruqt/frontend
gozilla → /Users/victornielsen/Documents/instruqt/gozilla
lab-hcl → /Users/victornielsen/Documents/instruqt/lab-hcl
lab-sdk → /Users/victornielsen/Documents/instruqt/lab-sdk
```

#### 3.3: Prepare Git State

```bash
cd {REPO_PATH}

# Ensure on main/master
git checkout main || git checkout master

# Pull latest
git pull origin main || git pull origin master

# Create ticket branch
# Format: {user}/{ticket-id}-{repo}-{short-description}
git checkout -b vic/{TICKET-ID}-{repo}-{description}
```

Branch naming:
- Lowercase ticket ID: `ram-310`
- Repo name: `frontend`, `gozilla`, etc.
- Description: kebab-case from ticket title, truncated

Example: `vic/ram-310-frontend-update-cloud-credentials-form`

#### 3.4: Spawn Implementation Sub-Agent

Based on the work type, spawn the appropriate agent:

**For Go code:**
Use the Task tool with `go-executor` patterns:
```
Implement the following ticket:

Ticket: {IDENTIFIER}
Title: {title}
Description: {description}

Follow TDD:
1. Write failing tests first
2. Implement to make tests pass
3. Refactor if needed

Ensure:
- All tests pass: go test ./...
- Linting passes: golangci-lint run
- Build succeeds: go build ./...
```

**For React/Frontend code:**
Use the Task tool with `react-executor` patterns:
```
Implement the following ticket:

Ticket: {IDENTIFIER}
Title: {title}
Description: {description}

Follow TDD:
1. Write failing tests first
2. Implement to make tests pass
3. Ensure types are correct

Ensure:
- All tests pass: npm test
- Types check: npm run typecheck
- Lint passes: npm run lint
- Build succeeds: npm run build
```

**For Protobuf/Schema changes:**
```
Update schema files as specified:

Ticket: {IDENTIFIER}
Changes needed: {description}

After changes:
- Regenerate code: buf generate
- Ensure build passes
```

#### 3.5: Verify CI Locally

Before creating PR, run local checks:

```bash
# Go repos
go test ./...
go build ./...
golangci-lint run

# Frontend
npm test
npm run typecheck
npm run lint
npm run build
```

If checks fail:
1. Attempt to fix issues
2. If stuck after 3 attempts, mark ticket as blocked

#### 3.6: Commit and Push

```bash
git add -A
git commit -m "$(cat <<'EOF'
{Commit message based on ticket}

Ticket: {IDENTIFIER}
EOF
)"

git push -u origin {branch-name}
```

#### 3.7: Create Draft PR

Use the `create-pr` skill patterns:

```bash
gh pr create --draft \
  --title "{IDENTIFIER}: {ticket title}" \
  --body "$(cat <<'EOF'
## Summary

{Brief description of changes}

## Linear Ticket

[{IDENTIFIER}](https://linear.app/instruqt/issue/{IDENTIFIER})

## Changes

- {Change 1}
- {Change 2}

## Test Plan

- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed
EOF
)"
```

Capture PR URL:
```bash
PR_URL=$(gh pr view --json url -q '.url')
```

#### 3.8: Spawn Review Sub-Agent

Spawn a sub-agent to review the PR:

```
Review this PR for issues:

PR URL: {PR_URL}
Ticket: {IDENTIFIER}

Use the react-review or equivalent patterns to identify:
- Correctness issues
- Performance concerns
- Security vulnerabilities
- Code style problems
- Missing tests

Provide a structured review.
```

#### 3.9: Post Review as PR Comment

```bash
gh pr comment {PR_NUMBER} --body "$(cat <<'EOF'
## Self-Review

{Review content from sub-agent}

### Summary

{Overall assessment}

### Issues Found

{List of issues if any}

### Recommendations

{Suggestions for improvement}
EOF
)"
```

#### 3.10: Assess Readiness

Based on review:

**If ready for human review:**
```bash
# Move ticket to "In Review"
linearis issues update {IDENTIFIER} --state "In Review"

# Mark PR ready for review
gh pr ready
```

**If blocked:**
```bash
# Move ticket to "Blocked"
linearis issues update {IDENTIFIER} --state "Blocked"

# Comment on PR with reason
gh pr comment {PR_NUMBER} --body "$(cat <<'EOF'
## Blocked

This PR is blocked due to:

{Reason from review}

Requires human intervention to resolve.
EOF
)"
```

### Phase 4: Parallel Execution Opportunities

#### Exploration Agents
For complex tickets, spawn exploration agents first:
```
Explore the codebase to understand:
- How {feature} is currently implemented
- What patterns are used for {area}
- What tests exist for {component}

Report findings for implementation planning.
```

#### Validation Agents
After implementation, spawn validation agents:
```
Validate the implementation:
- Run full test suite
- Check for regressions
- Verify edge cases mentioned in ticket

Report validation results.
```

#### Cross-Repo Coordination
If tickets span multiple repos:
1. Implement dependency order (backend before frontend)
2. Spawn agents for independent repos in parallel
3. Wait for blockers before continuing

### Phase 5: Progress Reporting

Maintain status throughout:

```markdown
## Implementation Progress

| Ticket | Status | PR | Notes |
|--------|--------|-----|-------|
| RAM-306 | Done | #203 | Merged |
| RAM-307 | In Review | #143 | Awaiting review |
| RAM-308 | In Progress | - | Implementing... |
| RAM-309 | Blocked | - | Waiting on RAM-308 |
| RAM-310 | Pending | - | Not started |
```

## Integration with Existing Skills

This skill leverages:

- **linear-reader**: Read ticket details
- **linear-updater**: Update ticket states
- **create-pr**: Create GitHub PRs
- **pr-description**: Generate PR descriptions
- **go-planner / go-executor**: Go implementation
- **react-planner / react-executor**: React implementation
- **react-review**: Code review patterns

## Input Formats

### Option 1: Milestone
```
/implementation-orchestrator --milestone "Cloud Credentials v1" --project "Cloud Accounts"
```

### Option 2: Ticket List
```
/implementation-orchestrator RAM-306 RAM-307 RAM-308 RAM-309 RAM-310
```

### Option 3: Project
```
/implementation-orchestrator --project "Cloud Accounts support in Labs"
```

## Configuration

Required:
- **User prefix**: For branch naming (e.g., "vic")
- **Repository paths**: Map of repo names to local paths

Optional:
- **Auto-merge**: Merge PRs that pass review (default: false)
- **Parallel limit**: Max concurrent implementations (default: 1)
- **Review threshold**: Issues count to auto-block (default: 3)

## Error Handling

### Implementation Fails
```
1. Log the error
2. Attempt fix (up to 3 times)
3. If still failing, mark ticket as Blocked
4. Add comment with error details
5. Continue with next unblocked ticket
```

### CI Fails
```
1. Read CI error output
2. Attempt to fix
3. If cannot fix, mark as Blocked
4. Include CI logs in PR comment
```

### Dependency Deadlock
If all remaining tickets are blocked:
```
1. Report deadlock situation
2. List blocked tickets and their blockers
3. Ask for human intervention
```

## Safety Rules

1. **Never force push** - Only regular pushes
2. **Always draft PRs first** - Mark ready only after review
3. **Preserve ticket history** - Add comments, don't delete
4. **Report all actions** - Log everything for traceability
5. **Human approval for merge** - Never auto-merge without permission

## Loop Prevention

Track iterations per ticket:
- Max 5 implementation attempts
- Max 3 CI fix attempts
- Max 2 review-fix cycles

If exceeded:
```bash
linearis issues update {IDENTIFIER} --state "Blocked"
gh pr comment {PR_NUMBER} --body "Blocked: Exceeded retry limits. Human review required."
```

## When to Use

Use `implementation-orchestrator` when:
- You have a sequence of tickets ready for implementation
- Tickets have clear dependencies (blocking relations)
- You want automated implementation with review

Prerequisites:
- Tickets created (use `design-to-tickets` first)
- Blocking relations established
- All tickets in "Backlog" or "Todo" state

## Example Session

```
User: /implementation-orchestrator RAM-306 RAM-307 RAM-308

Orchestrator:
1. Reading tickets...
   - RAM-306: Add protobuf enums (no blockers, ready)
   - RAM-307: Add protobuf messages (blocked by RAM-306)
   - RAM-308: Add RPC definition (blocked by RAM-307)

2. Starting RAM-306...
   - Moving to "In Progress"
   - Navigating to lab-hcl repo
   - Creating branch: vic/ram-306-lab-hcl-add-protobuf-enums
   - Spawning Go implementation agent...
   - Tests passing, CI green
   - Creating draft PR #203
   - Spawning review agent...
   - Review complete, no issues
   - Moving to "In Review", marking PR ready

3. RAM-306 complete. RAM-307 now unblocked.

4. Starting RAM-307...
   [continues...]
```
