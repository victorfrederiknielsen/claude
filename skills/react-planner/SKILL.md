---
name: react-planner
description: Creates detailed React implementation plans from Linear tickets or general React requirements. Use when planning React features, components, or UI work. Specializes in modern React patterns from the official docs.
allowed-tools: Read, Grep, Glob, AskUserQuestion
---

# React Implementation Planner

This skill creates comprehensive implementation plans for React-based work. It analyzes requirements (from Linear tickets or direct requests) and generates detailed, actionable plans following modern React best practices.

## Modern React Principles

This planner follows the latest React documentation and emphasizes:

### Event-Driven Architecture Over useEffect
- Prefer event handlers and direct state updates
- Only use useEffect for synchronization with external systems
- Avoid useEffect for data transformations or event handling

### Component Design
- Server Components by default (when applicable)
- Client Components only when needed (interactivity, browser APIs)
- Composition over configuration
- Colocation of related logic

### State Management
- Local state with useState for component-specific data
- Context for shared state (sparingly)
- Reducers for complex state logic
- Server state (React Query, SWR) for data fetching

## Instructions

### 1. Analyze Requirements

From the input (Linear ticket output or direct request), identify:
- **User-facing changes**: What does the user see/interact with?
- **Component structure**: New components or modifications to existing ones?
- **State requirements**: What data needs to be managed?
- **Data flow**: Where does data come from and how does it flow?
- **Side effects**: Any external system synchronization needed?

### 2. Explore Existing Codebase

Use Read, Grep, and Glob to understand:
- **Component patterns**: How are similar components structured?
- **State management approach**: What patterns are already in use?
- **Styling approach**: CSS modules, styled-components, Tailwind, etc.?
- **Testing patterns**: What test utilities and patterns exist?
- **File structure**: Where should new files be created?

Search for:
```bash
# Find similar components
grep -r "component-name" src/

# Find state management patterns
grep -r "useState\|useReducer\|useContext" src/

# Find data fetching patterns
grep -r "fetch\|axios\|useQuery" src/
```

### 3. Design Component Architecture

Plan the component hierarchy:
- **Container components**: Handle data and business logic
- **Presentation components**: Handle UI and display
- **Shared components**: Reusable across features
- **Hooks**: Extract reusable logic

Consider:
- Can this be a Server Component?
- What props does each component need?
- What state is local vs shared?
- What can be composed vs configured?

### 4. Plan State Management

For each piece of state, determine:
- **Location**: Where should it live? (local, context, server)
- **Updates**: What events trigger changes?
- **Derived state**: Can it be calculated during render?
- **Persistence**: Does it need to be saved/synced?

Avoid:
- Duplicating state
- Storing derived values in state
- Using useEffect for event-driven updates

### 5. Identify Side Effects

Only plan useEffect for:
- Syncing with external systems (WebSocket, DOM APIs)
- Subscribing to external data sources
- Setting up third-party libraries

For everything else, use:
- Event handlers for user actions
- Render-time calculations for derived data
- State setters for data transformations

### 6. Plan Testing and Documentation Strategy

Include tests for:
- **User interactions**: Click, type, submit
- **Visual states**: Loading, error, success
- **Accessibility**: ARIA labels, keyboard navigation
- **Edge cases**: Empty states, errors, boundaries

Include Storybook stories for:
- **Component variants**: Different prop combinations
- **Visual states**: Loading, error, success, empty states
- **Interactive examples**: Real-world usage scenarios
- **Accessibility testing**: With a11y addon
- **Design system documentation**: Component API and guidelines

### 7. Generate Implementation Plan

Create a structured plan with:

```markdown
# React Implementation Plan: {Feature Name}

## Overview
- **Type**: {New Feature/Bug Fix/Refactor}
- **Scope**: {Component changes, new components, etc.}
- **Complexity**: {Low/Medium/High}

## Requirements Analysis
{Summary of what needs to be built and why}

## Existing Codebase Patterns
- **Component structure**: {findings}
- **State management**: {patterns found}
- **Styling approach**: {approach}
- **Testing setup**: {tools and patterns}
- **Storybook setup**: {if Storybook is configured, note the version and addon patterns}

## Component Architecture

### New Components
1. `ComponentName` (src/path/ComponentName.tsx)
   - **Purpose**: {what it does}
   - **Type**: {Server/Client Component}
   - **Props**: {list of props with types}
   - **State**: {local state needed}
   - **Children**: {child components}

### Modified Components
1. `ExistingComponent` (src/path/ExistingComponent.tsx)
   - **Changes**: {what needs to change}
   - **Impact**: {what else is affected}

## State Management Plan

### Local State
- `componentState`: {purpose and location}
  - **Updated by**: {event handlers}
  - **Used by**: {components/logic}

### Shared State (if needed)
- `sharedState`: {purpose}
  - **Provider location**: {where}
  - **Consumers**: {which components}

### Server State (if needed)
- **Endpoint**: {API endpoint}
- **Fetching strategy**: {on mount, on demand, etc.}
- **Cache strategy**: {if applicable}

## Side Effects Plan

{List any useEffect needs with justification}
- **Effect**: {description}
  - **Why needed**: {synchronization requirement}
  - **Cleanup**: {cleanup logic needed}

## Event Handlers

{List of event handlers and their responsibilities}
- `handleAction`: {what it does}
  - **Triggered by**: {user action}
  - **Updates**: {state changes}
  - **Side effects**: {any API calls, etc.}

## Implementation Steps

1. **Set up component structure**
   - Create {Component} in src/path/
   - Define props interface
   - Set up basic render

2. **Implement state management**
   - Add useState for {state}
   - Create event handlers for {actions}
   - Wire up state updates

3. **Add UI and styling**
   - Implement component markup
   - Add styles following {approach}
   - Ensure responsive design

4. **Integrate with existing code**
   - Import in {parent component}
   - Pass props from {source}
   - Handle data flow

5. **Add error handling**
   - Error boundaries (if needed)
   - Form validation (if applicable)
   - Loading states

6. **Write tests**
   - Unit tests for {logic}
   - Integration tests for {user flows}
   - Accessibility tests

7. **Create Storybook stories**
   - Story for each major variant
   - Interactive controls for props
   - Documentation for usage
   - Accessibility checks

## File Changes

### New Files
- `src/path/Component.tsx` - Main component
- `src/path/Component.test.tsx` - Tests
- `src/path/Component.stories.tsx` - Storybook stories
- `src/path/types.ts` - TypeScript types (if complex)
- `src/path/hooks/useCustomHook.ts` - Custom hooks (if needed)

### Modified Files
- `src/path/ParentComponent.tsx` - Import and use new component
- `src/path/types.ts` - Add new type definitions
- `src/path/api/endpoints.ts` - Add API integration (if needed)

## Technical Considerations

### Performance
- {considerations like memoization, virtualization, etc.}

### Accessibility
- {ARIA labels, keyboard navigation, screen reader support}

### Browser Compatibility
- {any browser-specific concerns}

### Testing Strategy
- {what to test and how}

### Storybook Documentation
- {component variants to showcase}
- {interactive examples needed}
- {accessibility requirements}

## Potential Challenges
1. {Challenge}: {mitigation strategy}
2. {Challenge}: {mitigation strategy}

## Questions to Clarify
- {Any unclear requirements}
- {Design decisions needed}

## Next Steps
- Use the `react-executor` skill to implement this plan
- Review with team for feedback
```

## Planning Patterns by Task Type

### New Feature Component
1. Identify the feature's purpose and user interactions
2. Design component hierarchy (container + presentation)
3. Plan state and event handlers (avoid useEffect)
4. Map data flow from source to UI
5. Plan error states and loading states
6. Design test cases

### Bug Fix
1. Reproduce and understand the bug
2. Identify the root cause (state, render, side effect?)
3. Plan minimal fix following existing patterns
4. Add test to prevent regression
5. Verify no breaking changes

### Refactoring
1. Understand current implementation
2. Identify issues (performance, maintainability, patterns)
3. Plan incremental refactoring steps
4. Ensure tests pass at each step
5. Update related code if needed

### Performance Optimization
1. Profile and identify bottlenecks
2. Plan optimizations (memoization, lazy loading, etc.)
3. Avoid premature optimization
4. Measure impact
5. Balance complexity vs gains

## React Patterns to Favor

### ✅ Good Patterns
- Event handlers for user actions
- Derived state calculated during render
- Lifting state up for shared data
- Composition with children props
- Custom hooks for reusable logic
- Error boundaries for resilience

### ❌ Patterns to Avoid
- useEffect for event handling
- Storing derived state
- Prop drilling more than 2-3 levels
- Giant components with many responsibilities
- Complex useEffect chains
- Premature abstraction

## Notes

- Always check existing patterns before introducing new ones
- Favor simple solutions over clever ones
- Consider the whole feature, not just the happy path
- Plan for errors, loading, and edge cases from the start
- This skill only plans - use `react-executor` to implement
