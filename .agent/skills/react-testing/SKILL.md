---
name: react-testing
description: React testing guidelines with Vitest and React Testing Library. Use when writing tests for React components.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Edit
  - Write
---

# React Testing Guidelines (Vitest + Testing Library)

## Dependencies

```json
{
  "devDependencies": {
    "vitest": "^1.0.0",
    "@testing-library/react": "^14.0.0",
    "@testing-library/jest-dom": "^6.0.0",
    "@testing-library/user-event": "^14.0.0",
    "jsdom": "^23.0.0",
    "msw": "^2.0.0"
  }
}
```

## Vitest Config

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    globals: true,
  },
});
```

## Test Setup

```ts
// src/test/setup.ts
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach } from 'vitest';

afterEach(() => {
  cleanup();
});
```

## Component Testing

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { UserCard } from './UserCard';

describe('UserCard', () => {
  const mockUser = { id: '1', name: 'John', email: 'john@test.com' };

  it('renders user information', () => {
    render(<UserCard user={mockUser} />);
    
    expect(screen.getByText('John')).toBeInTheDocument();
    expect(screen.getByText('john@test.com')).toBeInTheDocument();
  });

  it('calls onEdit when edit button clicked', async () => {
    const onEdit = vi.fn();
    const user = userEvent.setup();
    
    render(<UserCard user={mockUser} onEdit={onEdit} />);
    await user.click(screen.getByRole('button', { name: /edit/i }));
    
    expect(onEdit).toHaveBeenCalledWith('1');
  });

  it('does not show edit button when onEdit not provided', () => {
    render(<UserCard user={mockUser} />);
    
    expect(screen.queryByRole('button', { name: /edit/i })).not.toBeInTheDocument();
  });
});
```

## Testing Async Components

```tsx
import { render, screen, waitFor } from '@testing-library/react';
import { UserList } from './UserList';

describe('UserList', () => {
  it('shows loading initially', () => {
    render(<UserList />);
    expect(screen.getByRole('status')).toBeInTheDocument();
  });

  it('renders users after loading', async () => {
    render(<UserList />);
    
    await waitFor(() => {
      expect(screen.getByText('John')).toBeInTheDocument();
    });
  });

  it('shows error message on failure', async () => {
    // Mock API to fail
    server.use(
      http.get('/api/users', () => {
        return new HttpResponse(null, { status: 500 });
      })
    );
    
    render(<UserList />);
    
    await waitFor(() => {
      expect(screen.getByText(/error/i)).toBeInTheDocument();
    });
  });
});
```

## MSW API Mocking

```ts
// src/test/mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: '1', name: 'John', email: 'john@test.com' },
      { id: '2', name: 'Jane', email: 'jane@test.com' },
    ]);
  }),
  
  http.post('/api/users', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ id: '3', ...body }, { status: 201 });
  }),
];

// src/test/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

## Hook Testing

```tsx
import { renderHook, waitFor } from '@testing-library/react';
import { useUsers } from './useUsers';

describe('useUsers', () => {
  it('returns users after loading', async () => {
    const { result } = renderHook(() => useUsers());
    
    expect(result.current.loading).toBe(true);
    
    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });
    
    expect(result.current.users).toHaveLength(2);
  });
});
```

## Form Testing

```tsx
describe('UserForm', () => {
  it('submits form with valid data', async () => {
    const onSubmit = vi.fn();
    const user = userEvent.setup();
    
    render(<UserForm onSubmit={onSubmit} />);
    
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/name/i), 'Test User');
    await user.click(screen.getByRole('button', { name: /submit/i }));
    
    expect(onSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      name: 'Test User',
    });
  });

  it('shows validation errors', async () => {
    const user = userEvent.setup();
    
    render(<UserForm onSubmit={vi.fn()} />);
    await user.click(screen.getByRole('button', { name: /submit/i }));
    
    expect(screen.getByText(/email required/i)).toBeInTheDocument();
    expect(screen.getByText(/name required/i)).toBeInTheDocument();
  });
});
```

## Coverage Requirements

- Minimum 80% line coverage
- 100% coverage for critical UI logic
- All user interactions tested
- Error states and edge cases covered

## Test Structure

```
Given (Arrange): Setup component with props/mocks
When (Act): User interaction or event
Then (Assert): Verify expected outcome
```

## Best Practices

- Query by role, label, or text (not test IDs)
- Use `userEvent` over `fireEvent`
- Prefer `findBy*` for async elements
- Test behavior, not implementation
- Mock API calls with MSW
- Keep tests focused and independent
- Use `describe` blocks to organize tests
