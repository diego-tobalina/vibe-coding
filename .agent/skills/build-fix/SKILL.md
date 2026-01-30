---
name: build-fix
description: Fix build and compilation errors systematically. Run when build fails.
disable-model-invocation: true
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Edit
---

# Fix Build Errors

Systematically resolve compilation/build errors.

## Process

1. **Run build** - Capture full error output
2. **Parse errors** - Extract and categorize all errors
3. **Prioritize** - Fix root causes first (dependency → syntax → type)
4. **Fix iteratively** - One error at a time, rebuild after each
5. **Verify** - Confirm clean build

## Build Commands

```bash
# Java/Maven
mvn compile 2>&1 | head -100

# Java/Gradle
./gradlew build 2>&1 | head -100

# TypeScript
npx tsc --noEmit 2>&1

# Node.js
npm run build 2>&1

# Full check
npm run build && npm run lint && npm run test
```

## Error Categories (Fix Order)

### 1. Dependency Errors
- Missing dependencies
- Version conflicts
- Circular dependencies

### 2. Import/Module Errors
- Missing imports
- Wrong import paths
- Module not found

### 3. Syntax Errors
- Typos
- Missing brackets/semicolons
- Invalid syntax

### 4. Type Errors
- Type mismatches
- Missing type annotations
- Incompatible types

### 5. Logic Errors
- Undefined variables
- Wrong method signatures
- Missing return statements

## Resolution Patterns

### Java
```java
// Missing import → Add import
import com.example.ClassName;

// Type mismatch → Fix type or cast
String value = object.toString();

// Null safety → Add null check
if (object != null) { ... }
```

### TypeScript
```typescript
// Missing type → Add type annotation
function process(data: DataType): ResultType { }

// Undefined check → Optional chaining
const value = obj?.property ?? defaultValue;

// Type assertion (last resort)
const typed = value as ExpectedType;
```

## Common Build Commands

```bash
# Java/Maven
mvn compile

# Java/Gradle
./gradlew build

# Node.js/TypeScript
npm run build
pnpm run build

# Check types only
npx tsc --noEmit
```

## Error Resolution Order

1. **Missing imports** - Add required dependencies
2. **Type errors** - Fix type mismatches
3. **Syntax errors** - Fix code syntax
4. **Dependency issues** - Update package versions

## Output Format

### Build Output
```
[paste build errors here]
```

### Errors Found: X

| # | File | Error | Fix Applied |
|---|------|-------|-------------|
| 1 | ... | ... | ... |

### Verification
```
[final build output - should be clean]
```

## Rules

- Always run build first to get current errors
- Fix one error at a time
- Rebuild after each fix to catch cascading issues
- Never suppress errors with @ts-ignore or similar
- If stuck, explain the error and ask for guidance

---

Run build command and show me the errors.
