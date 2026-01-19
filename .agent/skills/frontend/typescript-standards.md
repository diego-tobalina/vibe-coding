# Estándares TypeScript

## Configuración Estricta

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

## Tipos Básicos

```typescript
// Primitivos
const name: string = "John";
const age: number = 30;
const active: boolean = true;

// Arrays
const numbers: number[] = [1, 2, 3];
const names: Array<string> = ["a", "b"];

// Objetos
interface User {
  id: number;
  name: string;
  email?: string; // opcional
}

// Union Types
type Status = "pending" | "active" | "deleted";

// Literal Types
type Direction = "north" | "south" | "east" | "west";
```

## Funciones

```typescript
// Con tipos
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function
const multiply = (a: number, b: number): number => a * b;

// Optional params
function greet(name: string, greeting?: string): string {
  return `${greeting ?? "Hello"}, ${name}`;
}

// Default params
function createUser(name: string, role: string = "user"): User {
  return { name, role };
}
```

## Tipos Avanzados

```typescript
// Generics
interface Response<T> {
  data: T;
  status: number;
}

// Utility Types
type PartialUser = Partial<User>;      // Todos opcionales
type RequiredUser = Required<User>;    // Todos requeridos
type ReadonlyUser = Readonly<User>;    // Inmutable
type PickedUser = Pick<User, "id" | "name">;
type OmittedUser = Omit<User, "password">;

// Record
type UserMap = Record<string, User>;

// Mapped Types
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};
```

## Type Guards

```typescript
function isUser(obj: unknown): obj is User {
  return typeof obj === "object" 
    && obj !== null 
    && "id" in obj;
}

if (isUser(data)) {
  console.log(data.id); // TypeScript sabe que es User
}
```

## Buenas Prácticas

1. Preferir `interface` para objetos
2. Usar `type` para unions y utilitarios
3. Evitar `any` - usar `unknown`
4. Tipar retornos de función explícitamente
5. Usar readonly cuando sea posible
6. Preferir union types sobre enums
7. Usar const assertions para literales
