# Patrones React

## Estructura de Componente

```tsx
interface UserCardProps {
  user: User;
  onSelect?: (user: User) => void;
  className?: string;
}

export function UserCard({ user, onSelect, className }: UserCardProps) {
  return (
    <div className={cn("p-4 rounded-lg", className)}>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      {onSelect && (
        <button onClick={() => onSelect(user)}>Select</button>
      )}
    </div>
  );
}
```

## Hooks

### useState
```tsx
const [count, setCount] = useState(0);
const [user, setUser] = useState<User | null>(null);
```

### useEffect
```tsx
useEffect(() => {
  fetchData();
  
  return () => {
    // cleanup
  };
}, [dependency]);
```

### useMemo
```tsx
const sortedUsers = useMemo(
  () => users.sort((a, b) => a.name.localeCompare(b.name)),
  [users]
);
```

### useCallback
```tsx
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);
```

## Custom Hooks

```tsx
function useUser(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;

    async function fetch() {
      try {
        setLoading(true);
        const data = await userService.getById(userId);
        if (!cancelled) setUser(data);
      } catch (e) {
        if (!cancelled) setError(e as Error);
      } finally {
        if (!cancelled) setLoading(false);
      }
    }

    fetch();
    return () => { cancelled = true; };
  }, [userId]);

  return { user, loading, error };
}
```

## React Query

```tsx
function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => userService.getById(userId),
  });
}

function useUpdateUser() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: userService.update,
    onSuccess: (data) => {
      queryClient.invalidateQueries(['user', data.id]);
    },
  });
}
```

## Patterns

### Composition
```tsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

Card.Header = function Header({ children }) {
  return <div className="card-header">{children}</div>;
};

Card.Body = function Body({ children }) {
  return <div className="card-body">{children}</div>;
};

// Uso
<Card>
  <Card.Header>Title</Card.Header>
  <Card.Body>Content</Card.Body>
</Card>
```

### Render Props
```tsx
function DataFetcher({ url, render }) {
  const { data, loading } = useFetch(url);
  return render({ data, loading });
}

<DataFetcher 
  url="/api/users" 
  render={({ data }) => <UserList users={data} />} 
/>
```

### HOC (Higher-Order Component)
```tsx
function withAuth(Component: React.ComponentType) {
  return function AuthenticatedComponent(props: any) {
    const { user } = useAuth();
    if (!user) return <Redirect to="/login" />;
    return <Component {...props} />;
  };
}
```

## Mejores Prácticas

1. Componentes pequeños y enfocados
2. Props tipadas con interfaces
3. Evitar prop drilling (usar Context)
4. Memoizar cuando sea necesario
5. Keys únicas (nunca usar index)
6. Error boundaries para errores de render
7. Accessibility: ARIA, keyboard navigation
