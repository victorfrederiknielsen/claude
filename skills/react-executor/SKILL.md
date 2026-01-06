---
name: react-executor
description: Implements React features and components based on detailed plans. Use when you have a plan from react-planner and need to write the actual React code. Follows modern React best practices and the new React docs.
---

# React Implementation Executor

This skill implements React features based on detailed implementation plans. It follows modern React best practices from the official documentation, favoring event-driven patterns over useEffect and writing self-documenting code.

## Core Principles

### Modern React Patterns (New Docs)
- **Event-driven over useEffect**: Handle user actions with event handlers, not effects
- **Server Components by default**: Use Client Components only when needed
- **Derived state**: Calculate during render instead of storing in state
- **Composition**: Use children and props over configuration
- **Local state first**: Only lift state when actually shared

### Code Documentation Philosophy
- **NO comments for single-line changes**: Code should be self-documenting
- **DO document APIs and public functions**: Explain what, why, and edge cases
- **DO document complex logic**: Explain non-obvious algorithms or business rules
- **DO use descriptive names**: Variables and functions should explain themselves

### Code Style
- Use TypeScript for type safety
- Follow existing project patterns
- Prefer explicit over implicit
- Keep components focused and small
- Extract reusable logic to custom hooks

## Implementation Workflow

### 1. Review the Plan

Before starting implementation, carefully review:
- **Component architecture**: What components need to be created/modified?
- **State management**: Where does state live and how does it flow?
- **Event handlers**: What user actions need handling?
- **Side effects**: Any useEffect needed (should be minimal)?
- **File locations**: Where should new files be created?

### 2. Set Up Component Structure

Create components following the plan:

```typescript
// Good: Self-documenting component with clear purpose
interface WidgetCardProps {
  widget: Widget;
  onEdit: (id: string) => void;
  onDelete: (id: string) => void;
}

/**
 * Displays a widget card with edit and delete actions.
 * Handles user interactions and delegates state changes to parent.
 */
export function WidgetCard({ widget, onEdit, onDelete }: WidgetCardProps) {
  return (
    <Card>
      <h3>{widget.name}</h3>
      <p>{widget.description}</p>
      <Button onClick={() => onEdit(widget.id)}>Edit</Button>
      <Button onClick={() => onDelete(widget.id)}>Delete</Button>
    </Card>
  );
}
```

**What NOT to do:**
```typescript
// Bad: Unnecessary comments for obvious code
export function WidgetCard({ widget, onEdit, onDelete }: WidgetCardProps) {
  return (
    <Card>
      {/* Display widget name */}
      <h3>{widget.name}</h3>
      {/* Display widget description */}
      <p>{widget.description}</p>
      {/* Edit button */}
      <Button onClick={() => onEdit(widget.id)}>Edit</Button>
    </Card>
  );
}
```

### 3. Implement State Management

Follow the plan's state management approach:

**Local state for component-specific data:**
```typescript
/**
 * Form for creating or editing widgets.
 * Manages form state locally and notifies parent on submit.
 */
export function WidgetForm({ initialWidget, onSave }: WidgetFormProps) {
  const [name, setName] = useState(initialWidget?.name ?? '');
  const [description, setDescription] = useState(initialWidget?.description ?? '');

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    onSave({ name, description });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Widget name"
      />
      <textarea
        value={description}
        onChange={(e) => setDescription(e.target.value)}
        placeholder="Description"
      />
      <button type="submit">Save</button>
    </form>
  );
}
```

**Derived state (calculate during render):**
```typescript
/**
 * Displays a list of widgets with filtering and sorting.
 */
export function WidgetList({ widgets, filter }: WidgetListProps) {
  const filteredWidgets = widgets.filter(w =>
    w.name.toLowerCase().includes(filter.toLowerCase())
  );

  const sortedWidgets = [...filteredWidgets].sort((a, b) =>
    a.name.localeCompare(b.name)
  );

  return (
    <ul>
      {sortedWidgets.map(widget => (
        <li key={widget.id}>{widget.name}</li>
      ))}
    </ul>
  );
}
```

### 4. Implement Event Handlers (Not useEffect!)

Handle user interactions with event handlers:

```typescript
/**
 * Manages the widget dashboard with CRUD operations.
 */
export function WidgetDashboard() {
  const [widgets, setWidgets] = useState<Widget[]>([]);
  const [isCreating, setIsCreating] = useState(false);

  const handleCreate = (newWidget: Omit<Widget, 'id'>) => {
    const widget = { ...newWidget, id: generateId() };
    setWidgets(prev => [...prev, widget]);
    setIsCreating(false);
  };

  const handleDelete = (id: string) => {
    setWidgets(prev => prev.filter(w => w.id !== id));
  };

  const handleEdit = (id: string, updates: Partial<Widget>) => {
    setWidgets(prev => prev.map(w =>
      w.id === id ? { ...w, ...updates } : w
    ));
  };

  return (
    <div>
      <button onClick={() => setIsCreating(true)}>New Widget</button>
      {isCreating && (
        <WidgetForm onSave={handleCreate} />
      )}
      <WidgetList
        widgets={widgets}
        onEdit={handleEdit}
        onDelete={handleDelete}
      />
    </div>
  );
}
```

**What NOT to do:**
```typescript
// Bad: Using useEffect for event-driven updates
const handleCreate = (newWidget: Omit<Widget, 'id'>) => {
  setNewWidget(newWidget);
};

useEffect(() => {
  if (newWidget) {
    setWidgets(prev => [...prev, { ...newWidget, id: generateId() }]);
    setNewWidget(null);
  }
}, [newWidget]);
```

### 5. Use useEffect Only for External Synchronization

Only use useEffect for synchronizing with external systems:

```typescript
/**
 * Syncs widget data with browser localStorage.
 * Loads on mount and persists on changes.
 */
export function WidgetDashboard() {
  const [widgets, setWidgets] = useState<Widget[]>(() => {
    const saved = localStorage.getItem('widgets');
    return saved ? JSON.parse(saved) : [];
  });

  useEffect(() => {
    localStorage.setItem('widgets', JSON.stringify(widgets));
  }, [widgets]);

  // ... rest of component
}
```

```typescript
/**
 * Subscribes to real-time widget updates via WebSocket.
 */
export function useWidgetSubscription(userId: string) {
  const [widgets, setWidgets] = useState<Widget[]>([]);

  useEffect(() => {
    const ws = new WebSocket(`wss://api.example.com/widgets/${userId}`);

    ws.onmessage = (event) => {
      const update = JSON.parse(event.data);
      setWidgets(prev => applyUpdate(prev, update));
    };

    return () => ws.close();
  }, [userId]);

  return widgets;
}
```

### 6. Extract Reusable Logic to Custom Hooks

When logic is reused, extract to custom hooks:

```typescript
/**
 * Manages pagination state and provides navigation functions.
 *
 * @param totalItems - Total number of items to paginate
 * @param itemsPerPage - Number of items per page
 * @returns Current page, items for current page, and navigation functions
 */
export function usePagination<T>(items: T[], itemsPerPage: number = 10) {
  const [currentPage, setCurrentPage] = useState(1);

  const totalPages = Math.ceil(items.length / itemsPerPage);
  const startIndex = (currentPage - 1) * itemsPerPage;
  const currentItems = items.slice(startIndex, startIndex + itemsPerPage);

  const goToPage = (page: number) => {
    setCurrentPage(Math.max(1, Math.min(page, totalPages)));
  };

  const nextPage = () => goToPage(currentPage + 1);
  const prevPage = () => goToPage(currentPage - 1);

  return {
    currentPage,
    currentItems,
    totalPages,
    nextPage,
    prevPage,
    goToPage,
  };
}
```

### 7. Create Storybook Stories

Create comprehensive Storybook stories for component documentation and visual testing:

```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { WidgetCard } from './WidgetCard';

/**
 * WidgetCard displays a widget with edit and delete actions.
 *
 * Use this component to show widget information in a card layout
 * with interactive controls for managing the widget.
 */
const meta: Meta<typeof WidgetCard> = {
  title: 'Components/WidgetCard',
  component: WidgetCard,
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
  argTypes: {
    widget: {
      description: 'The widget data to display',
    },
    onEdit: { action: 'edit clicked' },
    onDelete: { action: 'delete clicked' },
  },
};

export default meta;
type Story = StoryObj<typeof WidgetCard>;

// Default story showing typical usage
export const Default: Story = {
  args: {
    widget: {
      id: '1',
      name: 'My Widget',
      description: 'A sample widget for demonstration',
    },
  },
};

// Story showing long content
export const LongContent: Story = {
  args: {
    widget: {
      id: '2',
      name: 'Widget with Very Long Name That Might Wrap',
      description: 'This is a widget with a very long description that demonstrates how the component handles text overflow and wrapping behavior in various layout scenarios.',
    },
  },
};

// Story showing minimal content
export const MinimalContent: Story = {
  args: {
    widget: {
      id: '3',
      name: 'Widget',
      description: 'Brief',
    },
  },
};

// Story showing interactive state
export const Interactive: Story = {
  args: {
    widget: {
      id: '4',
      name: 'Interactive Widget',
      description: 'Try clicking the edit and delete buttons',
    },
  },
  parameters: {
    docs: {
      description: {
        story: 'This story demonstrates the interactive behavior. Open the Actions panel to see event callbacks.',
      },
    },
  },
};
```

**For components with multiple variants:**

```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'danger'],
    },
    size: {
      control: 'select',
      options: ['sm', 'md', 'lg'],
    },
  },
};

export default meta;
type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Primary Button',
  },
};

export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Secondary Button',
  },
};

export const Danger: Story = {
  args: {
    variant: 'danger',
    children: 'Delete',
  },
};

export const Small: Story = {
  args: {
    size: 'sm',
    children: 'Small Button',
  },
};

export const Large: Story = {
  args: {
    size: 'lg',
    children: 'Large Button',
  },
};

export const Disabled: Story = {
  args: {
    disabled: true,
    children: 'Disabled Button',
  },
};

export const Loading: Story = {
  args: {
    isLoading: true,
    children: 'Loading...',
  },
};
```

**For complex components with state:**

```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { WidgetForm } from './WidgetForm';

const meta: Meta<typeof WidgetForm> = {
  title: 'Forms/WidgetForm',
  component: WidgetForm,
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
};

export default meta;
type Story = StoryObj<typeof WidgetForm>;

export const Empty: Story = {
  args: {
    onSave: (data) => console.log('Saved:', data),
  },
};

export const WithInitialData: Story = {
  args: {
    initialWidget: {
      name: 'Existing Widget',
      description: 'This widget already exists',
    },
    onSave: (data) => console.log('Updated:', data),
  },
};

export const WithValidationErrors: Story = {
  render: () => {
    const [errors, setErrors] = React.useState<ValidationErrors>({
      name: 'Name is required',
    });

    return (
      <WidgetForm
        onSave={(data) => {
          if (!data.name) {
            setErrors({ name: 'Name is required' });
          } else {
            setErrors({});
            console.log('Saved:', data);
          }
        }}
        errors={errors}
      />
    );
  },
};
```

**For components with data loading:**

```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { WidgetList } from './WidgetList';

const meta: Meta<typeof WidgetList> = {
  title: 'Components/WidgetList',
  component: WidgetList,
  tags: ['autodocs'],
};

export default meta;
type Story = StoryObj<typeof WidgetList>;

const mockWidgets = [
  { id: '1', name: 'Widget 1', description: 'First widget' },
  { id: '2', name: 'Widget 2', description: 'Second widget' },
  { id: '3', name: 'Widget 3', description: 'Third widget' },
];

export const Loaded: Story = {
  args: {
    widgets: mockWidgets,
    isLoading: false,
  },
};

export const Loading: Story = {
  args: {
    widgets: [],
    isLoading: true,
  },
};

export const Empty: Story = {
  args: {
    widgets: [],
    isLoading: false,
  },
  parameters: {
    docs: {
      description: {
        story: 'Shows the empty state when no widgets exist.',
      },
    },
  },
};

export const Error: Story = {
  args: {
    widgets: [],
    isLoading: false,
    error: new Error('Failed to load widgets'),
  },
};

export const ManyItems: Story = {
  args: {
    widgets: Array.from({ length: 50 }, (_, i) => ({
      id: `${i}`,
      name: `Widget ${i + 1}`,
      description: `Description for widget ${i + 1}`,
    })),
    isLoading: false,
  },
};
```

**Storybook Best Practices:**

- **One story per variant**: Create separate stories for each meaningful state
- **Use argTypes**: Define controls for interactive prop exploration
- **Add documentation**: Use JSDoc comments and story descriptions
- **Test edge cases**: Empty states, loading, errors, long content
- **Use actions**: Show event handlers with Storybook actions
- **Accessibility**: Use the a11y addon to check accessibility

### 8. Write Tests

Write tests following the project's testing patterns:

```typescript
/**
 * Tests for WidgetCard component
 */
describe('WidgetCard', () => {
  const mockWidget: Widget = {
    id: '1',
    name: 'Test Widget',
    description: 'A test widget',
  };

  it('displays widget information', () => {
    render(<WidgetCard widget={mockWidget} onEdit={vi.fn()} onDelete={vi.fn()} />);

    expect(screen.getByText('Test Widget')).toBeInTheDocument();
    expect(screen.getByText('A test widget')).toBeInTheDocument();
  });

  it('calls onEdit when edit button is clicked', () => {
    const handleEdit = vi.fn();
    render(<WidgetCard widget={mockWidget} onEdit={handleEdit} onDelete={vi.fn()} />);

    fireEvent.click(screen.getByText('Edit'));
    expect(handleEdit).toHaveBeenCalledWith('1');
  });

  it('calls onDelete when delete button is clicked', () => {
    const handleDelete = vi.fn();
    render(<WidgetCard widget={mockWidget} onEdit={vi.fn()} onDelete={handleDelete} />);

    fireEvent.click(screen.getByText('Delete'));
    expect(handleDelete).toHaveBeenCalledWith('1');
  });
});
```

### 9. Follow Existing Patterns

Always match the existing codebase:
- **File structure**: Create files in the same locations and with the same naming
- **Import style**: Use the same import patterns (relative vs absolute)
- **Component patterns**: Match existing component structures
- **Styling approach**: Use the same CSS/styling method
- **Testing approach**: Follow existing test patterns
- **Storybook patterns**: Match existing story structure and organization

## Documentation Guidelines

### DO Document These

**Public APIs and exported functions:**
```typescript
/**
 * Creates a new widget with the given attributes.
 *
 * @param attributes - Widget attributes (name and description)
 * @returns Promise resolving to the created widget
 * @throws {ValidationError} If attributes are invalid
 * @throws {NetworkError} If the API request fails
 */
export async function createWidget(
  attributes: WidgetAttributes
): Promise<Widget> {
  // implementation
}
```

**Complex logic or algorithms:**
```typescript
/**
 * Calculates optimal widget layout using a bin-packing algorithm.
 * Widgets are sorted by size (largest first) and placed in the first
 * container with sufficient space, creating new rows as needed.
 */
function calculateLayout(widgets: Widget[]): Layout {
  const sortedWidgets = [...widgets].sort((a, b) => b.size - a.size);
  // ... complex layout logic
}
```

**Non-obvious business rules:**
```typescript
/**
 * Determines if a user can edit a widget.
 * Users can edit if they are:
 * 1. The widget owner, OR
 * 2. An admin, OR
 * 3. In the widget's shared team AND the widget is not locked
 */
function canEditWidget(user: User, widget: Widget): boolean {
  if (widget.ownerId === user.id) return true;
  if (user.role === 'admin') return true;
  if (widget.isLocked) return false;
  return widget.sharedTeams.includes(user.teamId);
}
```

### DO NOT Document These

**Self-explanatory single-line operations:**
```typescript
// Bad: Obvious comment
const activeWidgets = widgets.filter(w => w.isActive); // Filter to active widgets only

// Good: No comment needed
const activeWidgets = widgets.filter(w => w.isActive);
```

**Standard React patterns:**
```typescript
// Bad: Documenting obvious React usage
const [count, setCount] = useState(0); // Initialize count state to 0

// Good: No comment needed
const [count, setCount] = useState(0);
```

**Obvious variable assignments:**
```typescript
// Bad: Unnecessary comments
const userName = user.name; // Get user name
const isEnabled = widget.status === 'enabled'; // Check if enabled

// Good: Self-documenting names
const userName = user.name;
const isEnabled = widget.status === 'enabled';
```

## Implementation Checklist

Before marking implementation complete:

- [ ] All components from the plan are created/modified
- [ ] State management follows event-driven patterns (no unnecessary useEffect)
- [ ] Event handlers are implemented correctly
- [ ] Only external synchronization uses useEffect
- [ ] Code is self-documenting (descriptive names, clear structure)
- [ ] Public APIs and complex logic are documented
- [ ] No comments on single-line obvious code
- [ ] Storybook stories are created for all new components
- [ ] Stories cover all major variants and states
- [ ] Stories include loading, error, and empty states
- [ ] Tests are written for key functionality
- [ ] Existing patterns are followed
- [ ] TypeScript types are properly defined
- [ ] Accessibility is considered (ARIA, keyboard navigation)
- [ ] Error states are handled
- [ ] Loading states are implemented

## Common Patterns

### Form Handling (Event-Driven)
```typescript
export function ContactForm({ onSubmit }: ContactFormProps) {
  const [formData, setFormData] = useState({ name: '', email: '' });

  const handleChange = (field: keyof typeof formData) => (
    e: ChangeEvent<HTMLInputElement>
  ) => {
    setFormData(prev => ({ ...prev, [field]: e.target.value }));
  };

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    onSubmit(formData);
    setFormData({ name: '', email: '' });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={formData.name} onChange={handleChange('name')} />
      <input value={formData.email} onChange={handleChange('email')} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### Modal/Dialog State
```typescript
export function Dashboard() {
  const [activeModal, setActiveModal] = useState<'create' | 'edit' | null>(null);
  const [selectedWidget, setSelectedWidget] = useState<Widget | null>(null);

  const openCreateModal = () => setActiveModal('create');
  const openEditModal = (widget: Widget) => {
    setSelectedWidget(widget);
    setActiveModal('edit');
  };
  const closeModal = () => {
    setActiveModal(null);
    setSelectedWidget(null);
  };

  return (
    <>
      <Button onClick={openCreateModal}>Create</Button>
      {activeModal === 'create' && <CreateModal onClose={closeModal} />}
      {activeModal === 'edit' && selectedWidget && (
        <EditModal widget={selectedWidget} onClose={closeModal} />
      )}
    </>
  );
}
```

### Data Fetching (Server State)
```typescript
/**
 * Fetches and manages widget data from the API.
 * Automatically refetches when dependencies change.
 */
export function useWidgets(userId: string) {
  const [widgets, setWidgets] = useState<Widget[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchWidgets() {
      try {
        setIsLoading(true);
        const data = await api.getWidgets(userId);
        if (!cancelled) {
          setWidgets(data);
          setError(null);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err as Error);
        }
      } finally {
        if (!cancelled) {
          setIsLoading(false);
        }
      }
    }

    fetchWidgets();
    return () => { cancelled = true; };
  }, [userId]);

  return { widgets, isLoading, error };
}
```

## Storybook Organization

### Story Naming Conventions

Follow these patterns for story organization:

**Title hierarchy:**
- `Components/ComponentName` - Reusable UI components
- `Forms/FormName` - Form components
- `Pages/PageName` - Full page components
- `Layouts/LayoutName` - Layout components
- `Features/FeatureName` - Feature-specific components

**Story names:**
- Use PascalCase for story names
- Name stories after their primary characteristic
- Common names: `Default`, `Loading`, `Error`, `Empty`, `Disabled`, `Active`, `Interactive`

### When to Create Stories

Create Storybook stories for:
- ✅ **Reusable components**: Buttons, cards, modals, inputs
- ✅ **Complex components**: Forms, data tables, charts
- ✅ **Design system components**: All components in the design system
- ✅ **Components with variants**: Different states, sizes, themes
- ✅ **Interactive components**: Tooltips, dropdowns, accordions

Skip stories for:
- ❌ **Page-specific one-offs**: Highly coupled to specific pages
- ❌ **Simple wrappers**: Trivial div containers with no logic
- ❌ **Internal implementation details**: Private sub-components

### Story File Location

Place story files next to their components:
```
src/
  components/
    Button/
      Button.tsx
      Button.test.tsx
      Button.stories.tsx  ← Stories here
    WidgetCard/
      WidgetCard.tsx
      WidgetCard.test.tsx
      WidgetCard.stories.tsx
```

## Notes

- Always favor event-driven patterns over useEffect
- Write self-documenting code, avoid obvious comments
- Document APIs, complex logic, and business rules
- Create comprehensive Storybook stories for reusable components
- Follow existing project patterns consistently
- Keep components small and focused
- Extract reusable logic to custom hooks
- Test user interactions, not implementation details
