---
name: linear-investigator
description: Investigates Linear tickets deeply to produce clarified requirements documents. Analyzes ticket details, attached images, codebase context, and asks clarifying questions. Produces comprehensive requirements documents ready for technical design. Use when you need to understand and clarify vague or incomplete requirements.
allowed-tools: Bash, Read, Grep, Glob, WebFetch, AskUserQuestion
---

# Linear Investigator Skill

This skill performs deep investigation of Linear tickets to transform vague or incomplete requirements into comprehensive, clarified requirements documents that are ready for technical design.

## Goals

- **Understand deeply**: Thoroughly analyze tickets, images, related issues, and codebase context
- **Clarify ambiguities**: Ask critical questions to resolve unclear requirements
- **Document thoroughly**: Create comprehensive requirements documents covering user needs, technical context, open questions, and design implications
- **Enable technical design**: Produce output that technical designers can immediately use to create implementation plans

## Prerequisites

The Linearis CLI tool must be installed and configured:

```bash
npm install -g --install-links czottmann/linearis
```

Authenticate with your Linear API token:
- Set `LINEAR_API_TOKEN` environment variable, OR
- Save token to `~/.linear_api_token` file

Get your API token from Linear: Settings → Security & Access → Personal API keys

## Workflow Overview

```
Linear Ticket URL
    ↓
1. Fetch ticket details (linearis)
    ↓
2. Analyze description, requirements, context
    ↓
3. Fetch and analyze attached images/files
    ↓
4. Explore related codebase areas
    ↓
5. Identify gaps and ambiguities
    ↓
6. Ask critical clarifying questions
    ↓
7. Generate comprehensive requirements document
```

## Instructions

### 1. Extract and Fetch Ticket

From the Linear URL, extract the ticket identifier:
- URL format: `https://linear.app/{workspace}/issue/{IDENTIFIER}/...`
- Example: `https://linear.app/instruqt/issue/RAA-993/...` → `RAA-993`

Fetch comprehensive ticket information:

```bash
linearis issues read {IDENTIFIER}
```

This returns JSON with:
- `identifier`: Ticket ID
- `title`: Ticket title
- `description`: Full description (markdown)
- `state`: Current state
- `priority`: Priority level
- `labels`: Array of labels
- `team`: Team information
- `assignee`: Assigned user
- `parentIssue`: Parent ticket
- `subIssues`: Array of subtasks
- `project`: Associated project
- `embeds`: **Attached files with download URLs**
- `comments`: Comments on the ticket

### 2. Analyze Ticket Content

Examine the ticket thoroughly to understand:

**User Requirements**
- What problem is being solved?
- Who are the end users?
- What are the user stories or use cases?
- What are the explicit acceptance criteria?
- What are the implicit expectations?

**Technical Scope**
- What parts of the system are affected? (Frontend, backend, database, etc.)
- Is this a new feature, bug fix, enhancement, or refactor?
- What is the estimated complexity?

**Context & Dependencies**
- Are there parent issues or related tickets?
- Are there comments providing additional context?
- Are there blocking issues or dependencies?
- Is there historical context needed?

### 3. Fetch and Analyze Attached Images/Files

**CRITICAL**: Always check the `embeds` field for attached files.

For each embed:
```bash
# Download the file
curl -o /tmp/ticket-image-{n}.{ext} "{downloadUrl}"
```

Then use the Read tool to analyze images:
```
Read the downloaded image file and analyze it for:
- UI mockups and design details
- Wireframes and user flows
- Architecture diagrams
- Error screenshots or bug reproductions
- Data models or schemas
- Any other visual information
```

**Document findings from images**:
- What UI elements are shown?
- What interactions are depicted?
- What states are visible? (loading, error, success)
- What data fields are present?
- Are there annotations or notes?
- What design patterns are evident?

### 4. Explore Codebase Context

Use Grep, Glob, and Read to understand the technical landscape:

**Find similar features**:
```bash
# Search for related components or patterns
Grep pattern="similar feature name" output_mode="files_with_matches"
```

**Understand existing patterns**:
- How are similar features implemented?
- What are the naming conventions?
- What are the architectural patterns?
- What libraries or frameworks are used?

**Identify affected areas**:
- Which files will likely need changes?
- What services or APIs exist?
- What database tables or models are involved?

### 5. Identify Gaps and Ambiguities

As you analyze, track:

**Missing Information**
- What requirements are unclear or incomplete?
- What edge cases are not addressed?
- What error scenarios are not specified?
- What performance requirements are missing?

**Technical Ambiguities**
- What architectural decisions need to be made?
- What integration points are unclear?
- What data flows are not specified?
- What security considerations are missing?

**Design Questions**
- What UI/UX decisions are open?
- What interaction patterns are undefined?
- What responsive behavior is expected?
- What accessibility requirements exist?

### 6. Ask Critical Clarifying Questions

Use the AskUserQuestion tool to resolve **critical** ambiguities that block understanding.

**Ask about**:
- **Must-haves vs nice-to-haves**: What is truly required for the first version?
- **User flows**: How should users interact with this feature?
- **Data sources**: Where does the data come from?
- **Error handling**: What should happen when things fail?
- **Integration points**: How does this connect to existing features?
- **Success metrics**: How will we know this is working well?

**Example questions**:
```
Should this feature work offline or require internet connectivity?

What should happen if the API call fails - show an error or fall back to cached data?

Is this feature available to all users or only specific roles?

Should data be persisted locally or only fetched from the server?
```

**Don't ask about**:
- Implementation details (those come later)
- Minor UI polish decisions
- Standard patterns (follow existing conventions)
- Technical choices that don't affect requirements

Document other questions in the output - don't ask everything upfront.

### 7. Generate Requirements Document

Create a comprehensive markdown document with the following structure:

```markdown
# Requirements Document: {IDENTIFIER} - {Title}

**Ticket:** [{IDENTIFIER}](ticket-url)
**Status:** {state}
**Priority:** {priority}
**Assignee:** {assignee or "Unassigned"}
**Generated:** {current date}

---

## Executive Summary

{2-3 sentence overview of what this ticket is about and why it matters}

---

## User Requirements

### Problem Statement

{Clear description of the problem being solved}

### User Stories

{List of user stories in the format: "As a [user type], I want to [action], so that [benefit]"}

Example:
- As a team member, I want to filter widgets by type, so that I can quickly find premium widgets
- As an admin, I want to see usage analytics, so that I can understand adoption

### Acceptance Criteria

{Explicit criteria that must be met for this to be considered complete}

Example:
- [ ] Users can create new widgets via a modal form
- [ ] Widget list displays all user's widgets
- [ ] Error states are handled gracefully
- [ ] Feature works on mobile and desktop

### Out of Scope

{Explicitly list what is NOT included in this work}

---

## Technical Context

### Affected Systems

{List which parts of the system are affected}

Example:
- **Frontend**: React components in `src/components/widgets/`
- **Backend**: Go API handlers in `internal/api/widgets/`
- **Database**: New `widgets` table
- **Authentication**: Requires logged-in user

### Existing Patterns

{Document relevant existing patterns found in the codebase}

Example:
- Similar feature: Dashboard cards follow pattern in `src/components/DashboardCard.tsx`
- API structure: RESTful endpoints following `/api/{resource}` convention
- State management: Using React Query for server state
- Error handling: Standard error boundary pattern

### Dependencies

{List technical dependencies}

Example:
- Depends on User Authentication system
- Requires Database migration
- May need new API endpoint for widget types
- Related ticket: RAA-123 (Widget types refactor)

### Technical Constraints

{Document any constraints or limitations}

Example:
- Must work with existing PostgreSQL database
- Cannot introduce new external dependencies without approval
- Must maintain backwards compatibility with API v2
- Performance: List endpoint must return in < 500ms for 10k records

---

## Image Analysis

{For each attached image, provide detailed analysis}

### Image 1: [Description from filename]

**Type:** {UI Mockup / Wireframe / Architecture Diagram / Screenshot / etc}

**Key Observations:**
- {Observation 1}
- {Observation 2}
- {Observation 3}

**UI Elements Identified:**
- {List of components, buttons, forms, etc}

**Interactions Shown:**
- {User interactions visible in the mockup}

**States Depicted:**
- {Loading, error, success, empty states, etc}

**Design Specifications:**
- Colors: {any colors shown}
- Layout: {grid, flex, responsive behavior}
- Typography: {any text styles}

**Questions Raised by Image:**
- {Any ambiguities or questions from the image}

{Repeat for each image}

---

## Open Questions

### Critical Questions (Need Answers Before Design)

{Questions that block technical design}

Example:
- ❓ Should widgets be shared between users or per-user only?
- ❓ What happens to widgets when a user is deactivated?
- ❓ Are there any rate limits on widget creation?

### Design Decisions Needed

{Architectural or design choices to be made}

Example:
- 🤔 Should we use GraphQL or REST for the widget API?
- 🤔 Should widget state be cached client-side or always fetched fresh?
- 🤔 Should we implement optimistic updates?

### Nice-to-Know (Can Decide During Implementation)

{Questions that can be answered during development}

Example:
- 💡 Should the widget list be paginated or infinite scroll?
- 💡 Should we add keyboard shortcuts for widget actions?
- 💡 What should the error message text say exactly?

---

## Design Implications

### Architectural Considerations

{High-level architecture thoughts}

Example:
- May need to introduce a new service layer for widget business logic
- Consider caching strategy for frequently accessed widgets
- May need event-driven updates if widgets are shared

### Integration Points

{Where this feature connects to other systems}

Example:
- Integrates with User service for ownership
- May need to integrate with Analytics for usage tracking
- Should integrate with Notification system for widget updates

### Data Model Implications

{Database or data structure considerations}

Example:
- New `widgets` table with columns: id, user_id, name, type, created_at
- Consider indexing on user_id and type for performance
- May need migration strategy for existing data

### Security Considerations

{Security and privacy implications}

Example:
- Must validate user owns widget before allowing modifications
- Need to sanitize widget names to prevent XSS
- Consider rate limiting on widget creation API

### Performance Considerations

{Performance implications and requirements}

Example:
- Widget list query may become slow with many users
- Consider pagination or virtual scrolling for large lists
- May need to implement caching to reduce API calls

### UI/UX Considerations

{User experience and interface considerations}

Example:
- Consider loading states during widget creation
- Need clear error messages for validation failures
- Should widgets be sortable or filterable?
- What happens when a user has no widgets (empty state)?

---

## Related Tickets & Context

### Parent/Child Tickets

{List related tickets in the hierarchy}

Example:
- **Parent:** RAA-100 - Widget Management Initiative
- **Children:**
  - RAA-201 - Widget Creation
  - RAA-202 - Widget List View
  - RAA-203 - Widget Details

### Related Issues

{Other relevant tickets}

Example:
- RAA-150 - User Dashboard Refactor (related UI patterns)
- RAA-175 - API Rate Limiting (may affect widget creation)

### Comments Summary

{Key points from ticket comments}

Example:
- PM noted in comment #5 that premium users need priority
- Designer shared updated mockups in comment #8
- Backend lead suggested using existing service pattern

---

## Assumptions

{Document assumptions being made}

Example:
- Assuming widgets are private to each user
- Assuming PostgreSQL database will be used
- Assuming React Query for data fetching
- Assuming mobile support is required

---

## Risks & Concerns

{Potential issues or risks}

Example:
- ⚠️ Database migration may take longer than expected
- ⚠️ API changes may break existing integrations
- ⚠️ Performance may degrade with large datasets
- ⚠️ Unclear if this conflicts with RAA-175 changes

---

## Next Steps

### Recommended Actions

1. {Action 1 - e.g., Get answers to critical questions}
2. {Action 2 - e.g., Create technical design document}
3. {Action 3 - e.g., Break down into implementation tickets}

### Ready for Technical Design?

**Status:** {✅ Ready / ⚠️ Needs clarification / ❌ Blocked}

{Explanation of readiness status}

---

## Appendix

### Linearis Output

<details>
<summary>Full ticket JSON (click to expand)</summary>

```json
{full JSON output from linearis}
```

</details>

### Codebase Files Referenced

{List of files examined during investigation}

- `path/to/file1.ts` - {brief note on why}
- `path/to/file2.tsx` - {brief note on why}

---

*This requirements document was generated by the linear-investigator skill to clarify requirements and provide context for technical design.*
```

## Output Format Guidelines

**Be comprehensive**: Include all relevant details from the ticket, images, and codebase exploration.

**Be specific**: Use concrete examples, file paths, and data structures.

**Be honest about unknowns**: Clearly mark what is uncertain or needs clarification.

**Be actionable**: Make it easy for someone to take this document and create a technical design.

**Use formatting**:
- ✅ for confirmed/ready items
- ❓ for critical questions
- 🤔 for design decisions
- 💡 for nice-to-know questions
- ⚠️ for risks/concerns
- 📸 for image-related content

## Example Investigation Flow

```
1. User provides: https://linear.app/instruqt/issue/RAA-456/add-widget-filtering

2. Fetch ticket:
   linearis issues read RAA-456

3. Analyze description:
   - User wants to filter widgets by type
   - No mockups attached
   - Parent issue mentions search functionality

4. Check for images:
   - No embeds found
   - Check parent issue for mockups

5. Explore codebase:
   - Find existing filter patterns in Dashboard
   - Locate widget list component
   - Identify API endpoint structure

6. Identify questions:
   - Should filtering be client-side or server-side?
   - What filter options: type only or also date/name?
   - Should filters persist across sessions?

7. Ask critical questions via AskUserQuestion:
   "Should widget filters be persistent across browser sessions?"
   "Is this client-side filtering or server-side?"

8. Generate comprehensive requirements document with:
   - User requirements (filter widgets efficiently)
   - Technical context (existing filter patterns found)
   - Open questions (documented with answers from user)
   - Design implications (may need API changes)
   - Next steps (ready for technical design)
```

## Tips for Effective Investigation

### Do
✅ Read the entire ticket description carefully
✅ Check all attached images thoroughly
✅ Look at parent/child tickets for context
✅ Explore similar features in the codebase
✅ Ask questions when requirements are ambiguous
✅ Document what you find AND what you don't find
✅ Be specific about technical details
✅ Consider edge cases and error scenarios

### Don't
❌ Skip image analysis even if images seem irrelevant
❌ Make assumptions without documenting them
❌ Propose implementation details (that's for later)
❌ Ignore comments on the ticket
❌ Forget to check for related tickets
❌ Be vague about unknowns
❌ Rush through the investigation

## When to Use This Skill

Use `linear-investigator` when:
- A ticket lacks clear requirements
- A ticket has images that need analysis
- A ticket needs context from the codebase
- You need to understand scope before planning
- Requirements need clarification before design
- A ticket is too vague to implement directly

**Do NOT use** for:
- Well-defined tickets ready for implementation (use `linear-planner`)
- Breaking tickets into subtasks (use `ticket-breakdown`)
- Actually implementing code (use `react-executor` or `go-executor`)

## Output Storage

After generating the requirements document, save it to:
```
{project-root}/docs/requirements/{IDENTIFIER}-requirements.md
```

Or if no docs directory exists:
```
{IDENTIFIER}-requirements.md
```

Then present the document to the user and ask if any sections need further clarification.

---

**Remember**: The goal is to transform unclear requirements into clear, comprehensive requirements documents that enable someone to create technical designs without needing to re-investigate the ticket.
