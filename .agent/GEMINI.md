---
name: global-coding-standards
description: Global coding standards and best practices for all projects. Applied across all workspaces.
mode: always_on
---

# Global Coding Standards

These rules apply to **all projects** across all workspaces in Antigravity.

---

## 1. Code Quality Principles

### Clarity Over Cleverness
- Write code that is easy to understand
- Avoid complex one-liners when simple code works
- Prefer explicit over implicit

### Fail Fast
- Validate inputs at function boundaries
- Throw meaningful exceptions early
- Never silently fail or swallow errors

### Configuration Over Magic
- Avoid hardcoded values
- Use parameters and environment variables
- Document configuration options

---

## 2. Naming Conventions (Universal)

| Type | Convention | Example |
|------|------------|---------|
| Variables | camelCase | `userEmail`, `orderTotal` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `API_TIMEOUT` |
| Functions | camelCase | `calculateTotal()`, `validateInput()` |
| Classes | PascalCase | `UserService`, `OrderProcessor` |
| Interfaces | PascalCase | `PaymentGateway`, `UserRepository` |
| Files | kebab-case or camelCase | `user-service.js`, `orderUtils.java` |
| Booleans | is/has/can prefix | `isValid`, `hasPermission`, `canDelete` |
| Collections | Plural nouns | `users`, `orderItems` |

---

## 3. Code Structure Limits

| Component | Max Size | Rationale |
|-----------|----------|-----------|
| Functions | 20 lines | Single responsibility |
| Classes | 400 lines | One concept per class |
| Files | 500 lines | Multiple classes split |
| Nesting | 4 levels max | Readability |
| Parameters | 4 max | Use objects/DTOs |

---

## 4. Comments & Documentation

**Comment WHY, not WHAT:**
```java
// ❌ WRONG
// Increment counter
counter++;

// ✅ CORRECT
// Retry 3 times because payment gateway occasionally times out
// See ticket PAY-1234 for context
int maxRetries = 3;
```

**Never leave commented-out code.** Use version control (git) instead.

---

## 5. Error Handling

### Always validate inputs:
```java
// ✅ CORRECT - Fail fast
if (order == null) {
    throw new IllegalArgumentException("Order cannot be null");
}
if (order.getItems().isEmpty()) {
    throw new IllegalArgumentException("Order must contain at least one item");
}
```

### Never catch generic Exception:
```java
// ❌ WRONG
try { process(); } catch (Exception e) { log.error("Error"); }

// ✅ CORRECT
try { process(); } 
catch (ValidationException e) { throw new ProcessingException("Invalid", e); }
```

---

## 6. Security Basics

- **Never** log passwords, tokens, or API keys
- Validate all user inputs
- Sanitize data before database queries
- Use parameterized queries (never string concatenation)
- Keep secrets in environment variables, never in code

---

## 7. Performance Awareness

### Database
- Use pagination for lists (default: 20 items)
- Avoid N+1 queries
- Index frequently queried columns

### Memory
- Don't load large collections entirely
- Use streaming for large datasets
- Close resources properly

### General
- Profile before optimizing
- Don't optimize prematurely

---

## 8. Git Workflow

### Commit Messages
Format: `type(scope): description`

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code restructuring
- `test`: Adding/modifying tests
- `docs`: Documentation
- `chore`: Maintenance

**Examples:**
- `feat(auth): add OAuth2 login`
- `fix(api): correct response status code`

### Branch Naming
- `main`: Production-ready
- `feature/description`: New features
- `fix/description`: Bug fixes

---

**Remember:** These are global standards. For project-specific rules, use workspace rules in `.agent/rules/`.
