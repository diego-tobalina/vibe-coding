# Core Guidelines for OpenCode

Universal principles for LLM-assisted development. These guidelines apply across all programming languages and project types.

**Philosophy:** Bias toward clarity and correctness over speed. For trivial scripts, move faster—but state this explicitly.

---

## 1. Think Before Coding
**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before writing any code:

### **What to do:**
- **State assumptions explicitly.** If inferring something not stated, say: *"I'm assuming X because Y—correct?"*
- **Present interpretations, don't pick silently.** If a request could mean two different things, show both options.
- **Advocate for simplicity.** If a simpler approach exists, say so.
- **Stop when confused.** Name what's unclear. Ask. Never proceed on shaky understanding.

---

## 2. Simplicity First
**Minimum viable code. No speculation.**

Write the **smallest, most obvious solution** that solves the stated problem:

### **The Rules:**
- ❌ No features beyond what was asked
- ❌ No abstractions for single-use code
- ❌ No "future-proofing" or configurability that wasn't requested
- ❌ No error handling for scenarios that can't happen

---

## 3. Surgical Changes
**Touch only what you must. Clean up only your own mess.**

When editing existing code, every changed line should trace directly to the user's request.

---

## 4. Goal-Driven Execution
**Define success criteria. Loop until verified.**

Transform vague tasks into **verifiable goals** before coding:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"

---

## 5. Verification Before Sharing
**Before presenting code, run this checklist:**

- [ ] **Diff audit:** What % of changed lines don't trace to the user's request? (Target: <5%)
- [ ] **Simplicity check:** Could this be half as long? If yes, revise.
- [ ] **Assumption check:** Did I state what I'm assuming?
- [ ] **Surgical check:** Did I touch code unrelated to the task?

---

## 6. Show Your Work (When It Matters)
**Complex logic needs inline rationale.**

For non-obvious decisions, add a comment explaining **why**, not what:

```java
// ✅ USEFUL - explains the reasoning
// Can't use batch updates due to MySQL 5.7 bug with CASE statements
// Processing individually prevents data corruption
for (user in users) { ... }
```

---

## 7. Configuration Over Magic
**Make behavior controllable. Avoid hidden defaults.**

```python
# ❌ BRITTLE (hard-coded assumption)
def read_file(path): return open(path).read()

# ✅ CONTROLLABLE
def read_file(path, encoding='utf-8'): return open(path, encoding=encoding).read()
```

---

## 8. Fail Fast and Loud
**Catch problems at the boundary, not deep in the stack.**

Validate inputs at function/method boundaries. Let the caller deal with bad data.

---

## Autonomous Coding Protocol
**For when you're coding without real-time user feedback.**

### Confidence Levels & Actions

| Confidence | Action |
|------------|--------|
| **95-100%** | Proceed confidently |
| **80-94%** | Proceed with safe default |
| **60-79%** | Proceed with conservative choice + note |
| **40-59%** | Conservative approach + detailed comment |
| **<40%** | STOP - cannot proceed autonomously |

### Safe Defaults
When unsure, always choose:
1. **Simplicity over complexity** (REST > GraphQL, Monolith > Microservices)
2. **Existing patterns over new ones** (match current codebase)
3. **Conservative over aggressive** (sync > async unless performance critical)
4. **Standard over custom** (JWT, bcrypt, UUIDs, ISO 8601)

### Red Lines: When to STOP Immediately
Never proceed autonomously if:
- 🛑 Requirements are contradictory
- 🛑 Security critical decision unclear
- 🛑 Multiple incompatible architectural approaches
- 🛑 Would require deleting/modifying significant existing code

---

## Available Skills in this Repository

Load the appropriate skill with: `@skill/<name>`

### Backend Development
- `@skill/java-coding` - Core Java + Lombok + modern features
- `@skill/springboot-coding` - Spring Boot + JPA + REST APIs
- `@skill/java-testing` - JUnit 5 + Mockito + AssertJ + Testcontainers
- `@skill/nodejs-coding` - Node.js + TypeScript + Express/Fastify
- `@skill/postgres-patterns` - PostgreSQL + JPA + Flyway
- `@skill/mongodb-patterns` - MongoDB + Spring Data MongoDB
- `@skill/redis-patterns` - Redis + caching + data structures

### API & Design
- `@skill/api-rest-design` - REST API standards (URLs, HTTP codes, errors, pagination)

### Frontend Development
- `@skill/react-coding` - React + TypeScript + Vite + Hooks
- `@skill/frontend-patterns` - Component patterns + composition
- `@skill/vercel-react-best-practices` - React performance + best practices

### DevOps & Infrastructure
- `@skill/docker` - Docker + Docker Compose + containerization

### Quality & Review
- `@skill/refactor` - Clean code + refactoring patterns
- `@skill/security-check` - Security audit + vulnerabilities
- `@skill/review` - Code review process + quality checks
- `@skill/tdd` - Test-Driven Development workflow
- `@skill/e2e` - End-to-end testing with Playwright/Cypress

### Tools & Utilities
- `@skill/build-fix` - Fix build and compilation errors
- `@skill/troubleshooting` - Debug common errors
- `@skill/logging` - Logging best practices (SLF4J, Logback, Pino)

---

## How to Use Multiple Skills

Unlike GitHub Copilot (single agent), **OpenCode allows combining multiple skills**:

```
User: "Create a Spring Boot service with tests"
You: Loading @skill/java-coding, @skill/springboot-coding, @skill/java-testing...
```

Or use the skill tool:
```
@skill/java-coding @skill/springboot-coding @skill/postgres-patterns
```

---

**Remember:** Your job is to solve the stated problem, not to anticipate every possible future need. When in doubt, do less.

---

## Coding Style Rules

### Naming Conventions (INFLEXIBLE)

| Type | Convention | Example | ❌ WRONG | ✅ CORRECT |
|------|------------|---------|----------|------------|
| Variable | camelCase | userEmail | user_email, UserEmail | userEmail |
| Constant | UPPER_SNAKE_CASE | MAX_RETRY_COUNT | maxRetryCount, MaxRetryCount | MAX_RETRY_COUNT |
| Function | camelCase (verb) | calculateTotal() | calc_total(), CalculateTotal() | calculateTotal() |
| Class | PascalCase | UserService | userService, user_service | UserService |
| Interface | PascalCase | PaymentProcessor | paymentProcessor, IPaymentProcessor | PaymentProcessor |
| Boolean | is/has/can prefix | isValid, hasPermission | valid, permission | isValid, hasPermission |
| Collection | Plural noun | users, orderItems | userList, orderItemArray | users, orderItems |

### Code Structure Rules

**Maximum Sizes (Hard Limits):**
| Component | Max Size | When to Split |
|-----------|----------|---------------|
| Function/Method | 20 lines | More than 1 responsibility |
| Class | 400 lines | More than 1 concept |
| File | 500 lines | Multiple classes |
| Nesting Level | 4 levels | Deeper nesting |
| Parameters | 4 params | Use builder/DTO |

### Error Handling Rules

**Fail Fast with Clear Messages:**
```java
// ❌ WRONG - Silent failure
if (order == null) { return; }

// ✅ CORRECT - Fail fast with context
if (order == null) {
    throw new IllegalArgumentException("Order cannot be null");
}
if (order.getItems().isEmpty()) {
    throw new IllegalArgumentException("Order must contain at least one item");
}
```

**Never catch generic Exception without rethrowing:**
```java
// ❌ WRONG - Swallows exception
try { processOrder(order); } catch (Exception e) { log.error("Error"); }

// ✅ CORRECT - Catch specific, wrap, and throw
try { processOrder(order); } 
catch (ValidationException e) { throw new OrderProcessingException("Invalid order", e); }
```

### Comments Rules

**Comment WHY, not WHAT:**
```java
// ❌ WRONG - Explains the obvious
// Increment counter
counter++;

// ✅ CORRECT - Explains why
// We retry 3 times because the payment gateway occasionally times out
// but succeeds on retry. See ticket PAY-1234.
int maxRetries = 3;
```

**Never leave commented-out code** - Use version control (git) instead.

---

## Git Workflow Rules

### Commit Messages

Format: `type(scope): description`

Types:
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code restructuring
- `test`: Adding/modifying tests
- `docs`: Documentation changes
- `chore`: Maintenance tasks

Examples:
- `feat(user): add email validation`
- `fix(order): correct price calculation`

### Branch Strategy

- `main`: Production-ready code
- `develop`: Integration branch
- `feature/*`: New features
- `fix/*`: Bug fixes

### Pull Request Guidelines

- Keep PRs small and focused
- Include tests for changes
- Update documentation if needed
- Request review before merge

---

## Performance Rules

### Database
- Use pagination for list queries
- Avoid N+1 queries (use JOIN FETCH or batch loading)
- Index frequently queried columns
- Use projections when full entity not needed

### Caching
- Cache frequently accessed, rarely changed data
- Set appropriate TTL
- Invalidate on writes
- Use cache-aside pattern

### Memory
- Avoid loading large collections into memory
- Use streaming for large datasets
- Close resources properly (try-with-resources)

### Frontend Performance (Core Web Vitals)

| Metric | Target | Description |
|--------|--------|-------------|
| **LCP** | < 2.5s | Largest Contentful Paint |
| **FID** | < 100ms | First Input Delay |
| **CLS** | < 0.1 | Cumulative Layout Shift |

### Profiling Before Optimizing

- Measure before optimizing
- Identify actual bottlenecks
- Don't optimize prematurely

**Tools:**
- Java: VisualVM, JProfiler, Async-profiler
- Node.js: Clinic.js, Chrome DevTools
- Frontend: Lighthouse, Chrome DevTools Performance tab
