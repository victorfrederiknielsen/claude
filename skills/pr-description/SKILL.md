---
name: pr-description
description: Write clear, concise PR descriptions in imperative mood for the current branch changes. Use when asked to write PR descriptions, generate PR text, or create pull request summaries.
---

# PR Description Writer

Generate professional pull request descriptions that clearly communicate what changes are being made and why.

## Core Principles

1. **Imperative Mood**: Use imperative verbs (Add, Fix, Update, Remove, Refactor) - NOT past tense (Added, Fixed)
2. **Be Specific**: Focus on what actually changed, not what you wish it did
3. **Be Honest**: Don't oversell - if it's a simple change, say so
4. **Be Clear**: Non-technical stakeholders should understand the "why"

## Workflow

### Step 1: Analyze the Changes

Use git commands to understand what's being changed:

```bash
# Get current branch name
git branch --show-current

# See what files changed
git diff --name-only origin/master...HEAD

# Get the actual diff
git diff origin/master...HEAD --stat

# See commit messages for context
git log origin/master..HEAD --oneline

# Get detailed diff for key files
git diff origin/master...HEAD
```

**Note:** Adjust the base branch if not `master` (could be `main`, `develop`, etc.)

### Step 2: Understand the Context

- Read the changed files to understand the full context
- Look at the commit messages for clues about intent
- Check if there are related issues or tickets mentioned
- Identify if this is a bug fix, feature, refactor, or technical improvement

### Step 3: Write the Description

Use this structure:

```markdown
## Summary
[1-2 sentences in imperative mood explaining WHAT and WHY]

## Changes
- [Imperative verb] [specific change]
- [Imperative verb] [specific change]
- [Imperative verb] [specific change]

## Testing Strategy
- [ ] [Specific test to run]
- [ ] [Specific scenario to verify]
- [ ] [Edge case to check]
```

## Writing Guidelines

### Imperative Mood Examples

✅ **Good:**
- Add user authentication middleware
- Fix race condition in payment processing
- Update dependencies to latest versions
- Remove deprecated API endpoints
- Refactor database query logic for performance

❌ **Bad:**
- Added user authentication middleware (past tense)
- Fixes race condition in payment processing (present tense)
- Updated dependencies to latest versions (past tense)
- I removed deprecated API endpoints (personal pronoun)
- This refactors database query logic (unnecessary words)

### Being Specific

✅ **Good:**
- Replace `react-helmet-async` with custom `useDocumentMeta` hook to reduce bundle size by ~15KB
- Fix Open Graph meta tags to use `property` attribute instead of `name` for social media compatibility
- Add cleanup function to remove stale meta tags on component unmount

❌ **Bad:**
- Improve performance (too vague)
- Fix some bugs (what bugs?)
- Make code better (how?)
- Enhance user experience (be specific how)
- Optimize everything (what specifically?)

### Not Over-Crediting

✅ **Good:**
- Update button color from blue to green per design spec
- Fix typo in error message
- Add missing null check to prevent crash

❌ **Bad:**
- Completely revolutionize the user interface with cutting-edge design
- Architect a robust, enterprise-grade error handling system
- Implement state-of-the-art null safety mechanisms

### Summary Section

The summary should:
- Start with an imperative verb
- Explain the main change in 1-2 sentences
- Include the "why" if it's not obvious
- Avoid jargon when possible

**Example:**
```
Replace the `react-helmet-async` dependency with lightweight custom hooks for managing document metadata. This reduces bundle size, simplifies the codebase, and fixes React Hooks rules violations in 9 files.
```

### Changes Section

- Use bullet points with imperative verbs
- Group related changes together
- Order by importance (most significant first)
- Include file paths for context when helpful
- Mention numbers when relevant (X files changed, Y lines added)

**Example:**
```
## Changes

### New Custom Hooks
- Add `useDocumentMeta` hook for managing title, description, and meta tags
- Add `useDocumentTitle` hook for simple title-only updates
- Include comprehensive test coverage for both hooks

### Migration
- Replace `<Helmet>` usage with custom hooks across 60+ components
- Remove `HelmetProvider` from App.tsx and Storybook
- Fix React Hooks rules violations by moving hooks before early returns

### Cleanup
- Remove `react-helmet-async` package dependency
- Remove `@types/react-helmet` dev dependency
```

### Testing Strategy Section

- Use checkboxes `- [ ]` for actionable items
- Be specific about what to test
- Include both happy path and edge cases
- Mention any manual testing steps
- Note if certain tests are automated

**Example:**
```
## Testing Strategy
- [ ] Verify all unit tests pass (31 tests in useDocumentMeta.spec.ts and useDocumentTitle.spec.ts)
- [ ] Check document title updates correctly when navigating between pages
- [ ] Test Open Graph tags using Facebook/LinkedIn sharing debugger
- [ ] Verify meta tags are cleaned up on component unmount (inspect DOM)
- [ ] Confirm no console errors in production build
```

## Additional Tips

1. **Link to Issues**: If there's a ticket, reference it: "Closes #123" or "Fixes RAA-456"
2. **Add Context**: If the change is subtle, explain why it matters
3. **Note Breaking Changes**: Clearly call out any breaking changes
4. **Include Screenshots**: For UI changes, mention if screenshots should be added
5. **Dependencies**: Note any new dependencies or version updates
6. **Migration Notes**: If changes require action from other devs, include a section

## Common Patterns

### Bug Fix PR
```markdown
## Summary
Fix race condition in checkout flow that causes duplicate charges when users click "Pay" multiple times rapidly.

## Changes
- Add loading state to prevent multiple payment submissions
- Debounce payment button clicks with 500ms delay
- Add transaction ID check before processing payment

## Testing Strategy
- [ ] Rapidly click payment button and verify only one charge is created
- [ ] Test with slow network connection (throttle to 3G)
- [ ] Verify error handling when payment provider is down
```

### Feature PR
```markdown
## Summary
Add dark mode support to the application with automatic system preference detection and manual toggle.

## Changes
- Add dark mode theme configuration with color tokens
- Implement `useDarkMode` hook with localStorage persistence
- Add theme toggle button to navigation header
- Update all components to support dark mode styles

## Testing Strategy
- [ ] Toggle dark mode manually and verify all pages render correctly
- [ ] Test automatic detection based on system preferences
- [ ] Verify theme preference persists across page reloads
- [ ] Check for any contrast/accessibility issues in dark mode
```

### Refactor PR
```markdown
## Summary
Refactor form validation logic to use Zod schema validation instead of custom validators, reducing code duplication and improving type safety.

## Changes
- Replace custom validation functions with Zod schemas
- Consolidate 5 separate validator files into 1 schema file
- Update form components to use Zod validation hooks
- Remove 200+ lines of manual validation code

## Testing Strategy
- [ ] Run existing form validation tests (all should pass)
- [ ] Test error messages display correctly for invalid inputs
- [ ] Verify form submission still works as expected
- [ ] Check that TypeScript types are properly inferred from schemas
```

## Edge Cases

### Very Small Changes
Don't over-complicate. Be brief:

```markdown
## Summary
Fix typo in user profile page header ("Accout" → "Account").

## Changes
- Correct spelling in ProfileHeader.tsx:45

## Testing Strategy
- [ ] Visual check that header displays "Account" correctly
```

### Multiple Unrelated Changes
Group by category:

```markdown
## Summary
Address multiple technical debt items including dependency updates, linting fixes, and test improvements.

## Changes

### Dependencies
- Update React from 18.2 to 18.3
- Update TypeScript from 5.1 to 5.2

### Code Quality
- Fix 15 ESLint warnings in auth module
- Add missing PropTypes to 3 components

### Tests
- Add test coverage for edge cases in parseDate utility
- Fix flaky timezone test

## Testing Strategy
- [ ] Run full test suite and verify all tests pass
- [ ] Check for any new TypeScript errors
- [ ] Smoke test authentication flow
```

---

Remember: The goal is clear communication, not impressive prose. Keep it simple, direct, and actionable.
