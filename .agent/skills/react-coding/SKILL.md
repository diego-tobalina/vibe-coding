---
name: react-coding
description: React coding guidelines with Vite, TypeScript, and modern patterns. Use when building React components and applications.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# React Coding Guidelines (Vite + TypeScript)

## Project Structure

```
src/
├── components/           # Reusable components
│   ├── ui/              # Basic UI components
│   └── common/          # Shared components
├── features/            # Feature-based modules
│   └── user/
│       ├── components/
│       ├── hooks/
│       ├── api.ts
│       └── types.ts
├── hooks/               # Global hooks
├── services/            # API services
├── stores/              # State management
├── types/               # Global types
├── utils/               # Utilities
└── App.tsx
```

## Component Patterns

### Functional Component
```tsx
interface UserCardProps {
  user: User;
  onEdit?: (id: string) => void;
}

export function UserCard({ user, onEdit }: UserCardProps) {
  const handleEdit = () => onEdit?.(user.id);
  
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      {onEdit && <button onClick={handleEdit}>Edit</button>}
    </div>
  );
}
```

### Component with State
```tsx
export function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetchUsers()
      .then(setUsers)
      .catch(e => setError(e.message))
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}><UserCard user={user} /></li>
      ))}
    </ul>
  );
}
```

## Custom Hooks

```tsx
function useUsers() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;
    
    fetchUsers()
      .then(data => !cancelled && setUsers(data))
      .catch(e => !cancelled && setError(e))
      .finally(() => !cancelled && setLoading(false));
    
    return () => { cancelled = true; };
  }, []);

  return { users, loading, error };
}
```

## API Service

```tsx
const API_BASE = import.meta.env.VITE_API_URL;

async function request<T>(path: string, options?: RequestInit): Promise<T> {
  const response = await fetch(`${API_BASE}${path}`, {
    headers: { 'Content-Type': 'application/json' },
    ...options,
  });
  
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }
  
  return response.json();
}

export const userApi = {
  getAll: () => request<User[]>('/users'),
  getById: (id: string) => request<User>(`/users/${id}`),
  create: (data: CreateUserDto) => 
    request<User>('/users', { method: 'POST', body: JSON.stringify(data) }),
  update: (id: string, data: UpdateUserDto) =>
    request<User>(`/users/${id}`, { method: 'PUT', body: JSON.stringify(data) }),
  delete: (id: string) => 
    request<void>(`/users/${id}`, { method: 'DELETE' }),
};
```

## TypeScript Types

```tsx
interface User {
  id: string;
  email: string;
  name: string;
  createdAt: string;
}

interface CreateUserDto {
  email: string;
  name: string;
}

type UpdateUserDto = Partial<CreateUserDto>;

interface ApiError {
  errorCode: string;
  message: string;
}
```

## Form Handling

```tsx
interface FormData {
  email: string;
  name: string;
}

export function UserForm({ onSubmit }: { onSubmit: (data: FormData) => void }) {
  const [formData, setFormData] = useState<FormData>({ email: '', name: '' });
  const [errors, setErrors] = useState<Partial<FormData>>({});

  const validate = (): boolean => {
    const newErrors: Partial<FormData> = {};
    if (!formData.email) newErrors.email = 'Email required';
    if (!formData.name) newErrors.name = 'Name required';
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (validate()) onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={formData.email}
        onChange={e => setFormData(prev => ({ ...prev, email: e.target.value }))}
      />
      {errors.email && <span className="error">{errors.email}</span>}
      
      <input
        type="text"
        value={formData.name}
        onChange={e => setFormData(prev => ({ ...prev, name: e.target.value }))}
      />
      {errors.name && <span className="error">{errors.name}</span>}
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Error Handling

### Error Boundary with Reset

```tsx
import { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, info: ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: ErrorInfo) {
    console.error('Error caught:', error, info);
    this.props.onError?.(error, info);
  }

  reset = () => {
    this.setState({ hasError: false, error: undefined });
  };

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={this.reset}>Try again</button>
        </div>
      );
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary fallback={<ErrorPage />}>
  <App />
</ErrorBoundary>
```

### Async Error Handling

```tsx
// Custom hook for async operations with error handling
function useAsync<T>(
  asyncFn: () => Promise<T>,
  deps: unknown[] = []
): {
  data: T | null;
  loading: boolean;
  error: Error | null;
  execute: () => Promise<void>;
  reset: () => void;
} {
  const [state, setState] = useState({
    data: null as T | null,
    loading: false,
    error: null as Error | null,
  });

  const execute = useCallback(async () => {
    setState(s => ({ ...s, loading: true, error: null }));
    
    try {
      const data = await asyncFn();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({ data: null, loading: false, error: error as Error });
    }
  }, deps);

  const reset = useCallback(() => {
    setState({ data: null, loading: false, error: null });
  }, []);

  return { ...state, execute, reset };
}

// Component using async error handling
export function UserProfile({ userId }: { userId: string }) {
  const { data: user, loading, error, execute, reset } = useAsync(
    () => fetchUser(userId),
    [userId]
  );

  useEffect(() => {
    execute();
  }, [execute]);

  if (loading) return <Spinner />;
  
  if (error) {
    return (
      <ErrorDisplay 
        error={error} 
        onRetry={execute}
        onReset={reset}
      />
    );
  }

  if (!user) return null;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### Error Display Component

```tsx
interface ErrorDisplayProps {
  error: Error;
  onRetry?: () => void;
  onReset?: () => void;
}

export function ErrorDisplay({ error, onRetry, onReset }: ErrorDisplayProps) {
  const isNetworkError = error.message.includes('network') || 
                         error.message.includes('fetch');
  
  return (
    <div className="error-display" role="alert">
      <h3>{isNetworkError ? 'Connection Error' : 'Error'}</h3>
      <p>{error.message}</p>
      
      {onRetry && (
        <button onClick={onRetry}>Try Again</button>
      )}
      
      {onReset && (
        <button onClick={onReset} className="secondary">
          Clear
        </button>
      )}
    </div>
  );
}
```

### Form Error Handling

```tsx
export function UserForm({ onSubmit }: { onSubmit: (data: FormData) => Promise<void> }) {
  const [formData, setFormData] = useState<FormData>({ email: '', name: '' });
  const [fieldErrors, setFieldErrors] = useState<Partial<FormData>>({});
  const [submitError, setSubmitError] = useState<string | null>(null);
  const [isSubmitting, setIsSubmitting] = useState(false);

  const validate = (): boolean => {
    const errors: Partial<FormData> = {};
    
    if (!formData.email) {
      errors.email = 'Email is required';
    } else if (!isValidEmail(formData.email)) {
      errors.email = 'Email is invalid';
    }
    
    if (!formData.name) {
      errors.name = 'Name is required';
    }
    
    setFieldErrors(errors);
    return Object.keys(errors).length === 0;
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setSubmitError(null);
    
    if (!validate()) return;
    
    setIsSubmitting(true);
    try {
      await onSubmit(formData);
      // Reset form on success
      setFormData({ email: '', name: '' });
    } catch (error) {
      setSubmitError(
        error instanceof Error ? error.message : 'An unexpected error occurred'
      );
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {submitError && (
        <div className="form-error" role="alert">
          {submitError}
        </div>
      )}
      
      <input
        type="email"
        value={formData.email}
        onChange={e => setFormData(prev => ({ ...prev, email: e.target.value }))}
        aria-invalid={!!fieldErrors.email}
        aria-describedby={fieldErrors.email ? 'email-error' : undefined}
      />
      {fieldErrors.email && (
        <span id="email-error" className="field-error" role="alert">
          {fieldErrors.email}
        </span>
      )}
      
      {/* Similar for name field */}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

## Loading & Error States

```tsx
interface AsyncState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

function useAsync<T>(asyncFn: () => Promise<T>, deps: unknown[] = []): AsyncState<T> {
  const [state, setState] = useState<AsyncState<T>>({
    data: null,
    loading: true,
    error: null,
  });

  useEffect(() => {
    let cancelled = false;
    setState(s => ({ ...s, loading: true }));
    
    asyncFn()
      .then(data => !cancelled && setState({ data, loading: false, error: null }))
      .catch(error => !cancelled && setState({ data: null, loading: false, error }));
    
    return () => { cancelled = true; };
  }, deps);

  return state;
}
```

## Common React Mistakes

### Mistake 1: Infinite useEffect Loop

**The Problem:**
```tsx
// ❌ WRONG - Infinite re-renders
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  });  // ❌ Missing dependency array!
  
  return <div>{user?.name}</div>;
}

// ❌ WRONG - Object in dependency array
function UserList({ filters }) {
  const [users, setUsers] = useState([]);
  
  useEffect(() => {
    fetchUsers(filters).then(setUsers);
  }, [filters]);  // ❌ filters object recreated every render!
  
  return ...;
}
```

**Why it happens:**
- No dependency array = runs after every render
- Objects/arrays in deps are recreated every render (new reference)
- State update triggers effect → updates state → triggers effect again

**Solutions:**
```tsx
// ✅ CORRECT - Empty deps = run once
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);  // Only re-run if userId changes
  
  return <div>{user?.name}</div>;
}

// ✅ CORRECT - Primitive deps only
function UserList({ category, status }) {
  const [users, setUsers] = useState([]);
  
  useEffect(() => {
    fetchUsers({ category, status }).then(setUsers);
  }, [category, status]);  // Primitives, stable references
  
  return ...;
}

// ✅ CORRECT - useMemo for stable object
function UserList({ filters }) {
  const [users, setUsers] = useState([]);
  
  // Memoize the filters object
  const stableFilters = useMemo(() => filters, [
    filters.category,
    filters.status
  ]);
  
  useEffect(() => {
    fetchUsers(stableFilters).then(setUsers);
  }, [stableFilters]);
  
  return ...;
}
```

### Mistake 2: Stale Closure

**The Problem:**
```tsx
// ❌ WRONG - Stale state in callback
function Counter() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    setCount(count + 1);  // ❌ count is stale!
  }, []);  // Empty deps = count is always 0
  
  return <button onClick={handleClick}>{count}</button>;
}

// ❌ WRONG - Stale state in setTimeout
function Message() {
  const [message, setMessage] = useState('Hello');
  
  useEffect(() => {
    const timer = setTimeout(() => {
      console.log(message);  // ❌ message is stale after 5 seconds
    }, 5000);
    return () => clearTimeout(timer);
  }, []);  // message never updates
  
  return <input value={message} onChange={e => setMessage(e.target.value)} />;
}
```

**Why it happens:**
- useCallback/useMemo with empty deps captures initial value
- Callbacks remember values from when they were created
- State changes but callback still uses old value

**Solutions:**
```tsx
// ✅ CORRECT - Functional updates
function Counter() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    setCount(c => c + 1);  // Uses current state, not stale closure
  }, []);  // No deps needed with functional update
  
  return <button onClick={handleClick}>{count}</button>;
}

// ✅ CORRECT - Include deps
function Message() {
  const [message, setMessage] = useState('Hello');
  
  useEffect(() => {
    const timer = setTimeout(() => {
      console.log(message);  // Always current value
    }, 5000);
    return () => clearTimeout(timer);
  }, [message]);  // Re-create timer when message changes
  
  return <input value={message} onChange={e => setMessage(e.target.value)} />;
}

// ✅ CORRECT - useRef for mutable values
function Message() {
  const [message, setMessage] = useState('Hello');
  const messageRef = useRef(message);
  
  // Keep ref updated
  messageRef.current = message;
  
  useEffect(() => {
    const timer = setTimeout(() => {
      console.log(messageRef.current);  // Always current via ref
    }, 5000);
    return () => clearTimeout(timer);
  }, []);  // No deps needed with ref
  
  return <input value={message} onChange={e => setMessage(e.target.value)} />;
}
```

### Mistake 3: Unnecessary Re-renders

**The Problem:**
```tsx
// ❌ WRONG - Parent re-render causes child re-render
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      <ExpensiveComponent data={getData()} />  // Re-renders even though data unchanged
    </div>
  );
}

function ExpensiveComponent({ data }) {
  // Heavy computation
  const processed = expensiveCalculation(data);
  return <div>{processed}</div>;
}
```

**Why it's bad:**
- Child re-renders when parent re-renders, even if props unchanged
- Objects/functions recreated every render (new reference)
- React sees different reference = re-render

**Solutions:**
```tsx
// ✅ CORRECT - useMemo for expensive calculations
function Parent() {
  const [count, setCount] = useState(0);
  
  const data = useMemo(() => getData(), []);  // Stable reference
  
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      <ExpensiveComponent data={data} />
    </div>
  );
}

// ✅ CORRECT - Memoize the component
const ExpensiveComponent = memo(function ExpensiveComponent({ data }) {
  const processed = useMemo(() => expensiveCalculation(data), [data]);
  return <div>{processed}</div>;
});

// ✅ CORRECT - useCallback for event handlers
function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');
  
  // Without useCallback, Child re-renders when text changes
  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []);  // Stable reference
  
  return (
    <div>
      <input value={text} onChange={e => setText(e.target.value)} />
      <Child onClick={handleClick} />
    </div>
  );
}

const Child = memo(function Child({ onClick }) {
  return <button onClick={onClick}>Click</button>;
});
```

### Mistake 4: Not Cleaning Up Effects

**The Problem:**
```tsx
// ❌ WRONG - Memory leak
function UserPresence({ userId }) {
  const [online, setOnline] = useState(false);
  
  useEffect(() => {
    // Subscribes to WebSocket
    websocket.subscribe(userId, (status) => {
      setOnline(status.online);
    });
    // ❌ No cleanup! Subscriptions accumulate
  }, [userId]);
  
  return <div>{online ? 'Online' : 'Offline'}</div>;
}

// ❌ WRONG - Event listener leak
function WindowSize() {
  const [width, setWidth] = useState(window.innerWidth);
  
  useEffect(() => {
    window.addEventListener('resize', () => {
      setWidth(window.innerWidth);
    });
    // ❌ Listener never removed!
  }, []);
  
  return <div>Width: {width}</div>;
}
```

**Why it's bad:**
- Subscriptions/event listeners accumulate
- Memory leaks
- Callbacks run on unmounted components (setState warnings)

**Solutions:**
```tsx
// ✅ CORRECT - Cleanup subscriptions
function UserPresence({ userId }) {
  const [online, setOnline] = useState(false);
  
  useEffect(() => {
    const unsubscribe = websocket.subscribe(userId, (status) => {
      setOnline(status.online);
    });
    
    return () => {
      unsubscribe();  // Cleanup on unmount or userId change
    };
  }, [userId]);
  
  return <div>{online ? 'Online' : 'Offline'}</div>;
}

// ✅ CORRECT - Remove event listeners
function WindowSize() {
  const [width, setWidth] = useState(window.innerWidth);
  
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    
    window.addEventListener('resize', handleResize);
    
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);
  
  return <div>Width: {width}</div>;
}

// ✅ CORRECT - AbortController for fetch
function UserData({ userId }) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    fetch(`/api/users/${userId}`, { signal: controller.signal })
      .then(res => res.json())
      .then(setData)
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error(err);
        }
      });
    
    return () => {
      controller.abort();  // Cancel fetch on cleanup
    };
  }, [userId]);
  
  return <div>{data?.name}</div>;
}
```

### Mistake 5: Incorrect Key Prop Usage

**The Problem:**
```tsx
// ❌ WRONG - Using index as key
function UserList({ users }) {
  return (
    <ul>
      {users.map((user, index) => (
        <li key={index}>  {/* ❌ Index changes when list reorders! */}
          {user.name}
        </li>
      ))}
    </ul>
  );
}

// ❌ WRONG - Math.random as key
function ItemList() {
  return (
    <ul>
      {items.map(item => (
        <li key={Math.random()}>  {/* ❌ New key every render! */}
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

**Why it's bad:**
- Index as key causes issues when list reorders
- Math.random causes complete re-render every time
- React uses key to identify elements - wrong key = bugs

**Solutions:**
```tsx
// ✅ CORRECT - Stable unique ID
function UserList({ users }) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>  {/* Stable, unique ID */}
          {user.name}
        </li>
      ))}
    </ul>
  );
}

// ✅ CORRECT - Composite key if needed
function CommentList({ comments }) {
  return (
    <ul>
      {comments.map(comment => (
        <li key={`${comment.postId}-${comment.id}`}>
          {comment.text}
        </li>
      ))}
    </ul>
  );
}

// ✅ CORRECT - Index only as last resort
function SimpleList({ items }) {
  // Only use index if items never reorder, filter, or insert
  return (
    <ul>
      {items.map((item, index) => (
        <li key={item.id || index}>  {/* Prefer ID, fallback to index */}
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

### Mistake 6: Conditional Hook Calls

**The Problem:**
```tsx
// ❌ WRONG - Conditional hook (violates React rules)
function UserProfile({ userId, isAdmin }) {
  if (!userId) {
    return <div>No user</div>;
  }
  
  const [user, setUser] = useState(null);  // ❌ Hook called conditionally!
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  if (isAdmin) {
    const [permissions, setPermissions] = useState([]);  // ❌ Conditional hook!
  }
  
  return <div>{user?.name}</div>;
}
```

**Why it breaks:**
- React relies on hook call order to track state
- Conditional calls break the order
- Leads to inconsistent state between renders

**Solutions:**
```tsx
// ✅ CORRECT - All hooks at top level
function UserProfile({ userId, isAdmin }) {
  // All hooks first
  const [user, setUser] = useState(null);
  const [permissions, setPermissions] = useState([]);
  
  // Effects next
  useEffect(() => {
    if (!userId) return;
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  useEffect(() => {
    if (!isAdmin || !userId) return;
    fetchPermissions(userId).then(setPermissions);
  }, [isAdmin, userId]);
  
  // Early return after all hooks
  if (!userId) {
    return <div>No user</div>;
  }
  
  return (
    <div>
      {user?.name}
      {isAdmin && <AdminPanel permissions={permissions} />}
    </div>
  );
}

// ✅ CORRECT - Early return component
function UserProfile({ userId, isAdmin }) {
  if (!userId) {
    return <div>No user</div>;
  }
  
  return <UserProfileContent userId={userId} isAdmin={isAdmin} />;
}

function UserProfileContent({ userId, isAdmin }: { userId: string, isAdmin: boolean }) {
  // Safe to use hooks here - userId is guaranteed to exist
  const [user, setUser] = useState(null);
  // ... hooks
  
  return <div>{user?.name}</div>;
}
```

---

## Best Practices

### Do's and Don'ts

| ✅ DO | ❌ DON'T |
|------|---------|
| Use TypeScript for all components | Use any type |
| Keep hooks at top level | Call hooks conditionally |
| Include all dependencies in useEffect | Forget dependency array |
| Clean up effects (subscriptions, listeners) | Leave resources open |
| Use useMemo for expensive calculations | Premature optimization |
| Use useCallback for stable references | Memoize everything |
| Use stable IDs for keys | Use index or Math.random |
| Split large components | Create 500-line components |
| Use functional state updates | Reference state directly in async callbacks |
| Colocate related files by feature | Organize by type (all components together) |

### Performance Rules

1. **Don't optimize prematurely** - Measure first with React DevTools Profiler
2. **Memoization has cost** - Only use when profiling shows benefit
3. **Props comparison cost** - memo() adds overhead, use wisely
4. **State updates batch** - Multiple setState calls in event handler batch automatically
5. **Context splits** - Split context to prevent unnecessary re-renders

### Component Size Guidelines

- **Ideal:** < 100 lines
- **Acceptable:** < 200 lines
- **Refactor needed:** > 300 lines

If component grows too large:
- Extract hooks
- Extract child components
- Extract utility functions
- Consider splitting features

### State Management Decision Tree

```
Do you need to share state?
├── NO → useState or useReducer
│         └── Is state complex?
│               ├── NO → useState
│               └── YES → useReducer
└── YES → How many components?
          ├── Few siblings → Lift state up
          ├── Many components → Context
          └── Complex app → Zustand/Redux
```

---

## Common AI Coding Mistakes (Autonomous Mode)

**When coding without user feedback, avoid these AI-specific errors:**

### 1. useEffect Dependency Hell

**The Mistake:** Forgetting dependencies or adding wrong ones
```tsx
// ❌ WRONG - Missing deps
useEffect(() => {
  fetchUser(userId).then(setUser);
}, []);  // Missing userId!

// ❌ WRONG - Object in deps
useEffect(() => {
  fetchUsers(filters);
}, [filters]);  // New object every render = infinite loop!

// ✅ CORRECT - Proper deps
useEffect(() => {
  fetchUser(userId).then(setUser);
}, [userId]);

// ✅ CORRECT - Memoize objects
const stableFilters = useMemo(() => filters, [filters.category]);
useEffect(() => {
  fetchUsers(stableFilters);
}, [stableFilters]);
```

### 2. Stale Closure in Event Handlers

**The Mistake:** Using old state values in callbacks
```tsx
// ❌ WRONG - Stale closure
const handleClick = useCallback(() => {
  console.log(count);  // Always logs initial value!
}, []);  // Empty deps

// ✅ CORRECT - Include deps
const handleClick = useCallback(() => {
  console.log(count);
}, [count]);

// ✅ CORRECT - Functional update
const handleClick = useCallback(() => {
  setCount(c => c + 1);  // Uses current value
}, []);
```

### 3. Forgetting Cleanup Functions

**The Mistake:** Not cleaning up subscriptions/listeners
```tsx
// ❌ WRONG - Memory leak
useEffect(() => {
  window.addEventListener('resize', handleResize);
  // No cleanup!
}, []);

// ✅ CORRECT - Cleanup
useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

### 4. Conditional Hook Calls

**The Mistake:** Calling hooks conditionally (breaks React rules)
```tsx
// ❌ WRONG - Conditional hook
if (user) {
  useEffect(() => {  // ERROR!
    fetchData();
  }, []);
}

// ✅ CORRECT - Always at top level
useEffect(() => {
  if (user) {
    fetchData();
  }
}, [user]);
```

### 5. Inline Object/Function Definitions

**The Mistake:** Creating new objects/functions in render
```tsx
// ❌ WRONG - New object every render
<ChildComponent config={{ theme: 'dark' }} />  // Child re-renders!

// ✅ CORRECT - Stable reference
const config = useMemo(() => ({ theme: 'dark' }), []);
<ChildComponent config={config} />
```

### 6. Autonomous Decision Checklist

**Before generating code, verify:**

- [ ] All hooks are at top level (no conditionals)
- [ ] useEffect has correct dependency array
- [ ] No objects/arrays in dependency arrays (unless memoized)
- [ ] Event handlers don't have stale closures
- [ ] Cleanup functions for all subscriptions
- [ ] Keys are stable (not index or Math.random)
- [ ] No conditional hook calls

**When uncertain about a hook:**
```tsx
// Add a comment documenting uncertainty
// UNCERTAIN: Not sure if this dependency is correct
// ASSUMPTION: Including 'userId' because fetch depends on it
// REVIEW: Verify this doesn't cause unnecessary refetches
useEffect(() => {
  fetchUser(userId);
}, [userId]);  // eslint-disable-line react-hooks/exhaustive-deps
```
