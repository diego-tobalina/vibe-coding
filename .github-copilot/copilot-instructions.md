# Core Guidelines for GitHub Copilot

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

### **Why this matters:**
When you assume without stating, you might build the wrong thing. The user then has to correct you, wasting both your time. Explicit assumptions let the user catch misunderstandings early.

### **Common Mistake:**
```
❌ User: "Add user authentication"
   You: [writes OAuth2 implementation with JWT, Redis, refresh tokens]
   
   Result: User wanted simple session-based auth for an internal tool.
   
✅ User: "Add user authentication"  
   You: "I see a few options:
         1. Simple session-based (cookies) - easier, for internal tools
         2. JWT tokens - stateless, good for APIs
         3. OAuth2 - for social login (Google, GitHub)
         Which fits your use case?"
```

**Test:** If your response starts with code, you probably skipped this step.

---

## 2. Simplicity First
**Minimum viable code. No speculation.**

Write the **smallest, most obvious solution** that solves the stated problem:

### **The Rules:**
- ❌ No features beyond what was asked
- ❌ No abstractions for single-use code
- ❌ No "future-proofing" or configurability that wasn't requested
- ❌ No error handling for scenarios that can't happen
- ❌ No defensive comments explaining "why we might need this later"

### **Why simple is better:**
1. Less code = fewer bugs
2. Easier to understand and modify later
3. Faster to write and test
4. Clearer intent

### **Example - What NOT to do:**
```java
// ❌ OVER-ENGINEERED
// User asked: "Create a method to calculate discount"

public interface DiscountStrategy {
    BigDecimal calculateDiscount(Order order);
}

@Component
public class DiscountCalculator {
    private final List<DiscountStrategy> strategies;
    private final DiscountConfig config;
    private final MetricsCollector metrics;
    
    public BigDecimal calculate(Order order, String strategyName) {
        metrics.recordCalculationStart();
        
        DiscountStrategy strategy = strategies.stream()
            .filter(s -> s.getClass().getSimpleName().equalsIgnoreCase(strategyName))
            .findFirst()
            .orElseThrow(() -> new StrategyNotFoundException(strategyName));
            
        BigDecimal discount = strategy.calculateDiscount(order);
        
        metrics.recordCalculationEnd(discount);
        return discount;
    }
}
```

### **Example - What TO do:**
```java
// ✅ SIMPLE AND CORRECT
// User asked: "Create a method to calculate discount"

public BigDecimal calculateDiscount(Order order) {
    if (order.getTotal().compareTo(new BigDecimal("100")) >= 0) {
        return order.getTotal().multiply(new BigDecimal("0.10")); // 10% off over $100
    }
    return BigDecimal.ZERO;
}
```

**Self-check:** If you wrote 200 lines and it could be 50, rewrite it.  
**Gold standard:** Would a senior engineer say "this is overcomplicated"? If yes, simplify.

---

## 3. Surgical Changes
**Touch only what you must. Clean up only your own mess.**

When editing existing code:

### **Don't touch:**
- Adjacent code that "could be better"
- Comments or formatting (match existing style, even if inconsistent)
- Dead code that was already there
- Unrelated methods, imports, or fields

### **Do clean up:**
- Imports/variables/methods that **your changes** made unused
- Inconsistencies **you introduced**

### **The test:**
Every changed line should trace directly to the user's request.

### **Example:**
```diff
# User asks: "Add email validation"

# ❌ INVASIVE (don't do this):
  def validate_user(data):
-     # check name
-     if data.name is None:
+     if not data.name:  # ← unsolicited "improvement" (changes logic to catch empty strings too)
          return False
+     if not is_valid_email(data.email):  # ← requested
+         return False
      return True

# ✅ SURGICAL (do this):
  def validate_user(data):
      # check name
      if data.name is None:
          return False
+     if not is_valid_email(data.email):  # ← only the requested change
+         return False
      return True
```

### **Why this matters:**
Every change you make that wasn't requested:
1. Increases risk of breaking something
2. Makes code review harder (distracts from the actual change)
3. Can introduce bugs in unrelated areas
4. Violates the user's expectations

---

## 4. Goal-Driven Execution
**Define success criteria. Loop until verified.**

Transform vague tasks into **verifiable goals** before coding:

| Vague request | Concrete goal |
|--------------|---------------|
| "Add validation" | "Write tests for invalid inputs, then make them pass" |
| "Fix the bug" | "Write a test that reproduces it, then make it pass" |
| "Refactor X" | "Ensure all tests pass before and after" |
| "Improve performance" | "Benchmark current speed, optimize, verify improvement" |

### **For multi-step tasks, state a verification plan:**

```
Plan:
1. Add email validation method → verify: unit tests pass
2. Integrate into user registration → verify: invalid emails rejected
3. Update error messages → verify: UI shows helpful feedback

Proceeding with step 1...
```

### **Why this matters:**
Strong success criteria enable independent work. When you define what "done" means:
- You know when to stop (prevents over-engineering)
- The user knows what to expect
- You can verify your work without guessing
- Weak criteria ("make it work") require constant back-and-forth

---

## 5. Verification Before Sharing
**Before presenting code, run this checklist:**

- [ ] **Diff audit:** What % of changed lines don't trace to the user's request? (Target: <5%)
- [ ] **Simplicity check:** Could this be half as long? If yes, revise.
- [ ] **Assumption check:** Did I state what I'm assuming, or did I silently decide?
- [ ] **Surgical check:** Did I touch code unrelated to the task?

If you answer "yes" to the last two, revise before sharing.

### **Common Mistakes to Check:**

1. **Did I remove debug code?**
   ```javascript
   // ❌ Don't leave this in production code
   console.log("DEBUG: user data =", userData);
   ```

2. **Did I handle all error cases?**
   ```java
   // ❌ This will throw NullPointerException if user is null
   String name = user.getName();
   
   // ✅ Defensive check
   String name = user != null ? user.getName() : "Anonymous";
   ```

3. **Are my imports clean?**
   ```java
   // ❌ Remove unused imports
   import java.util.Date;  // Not used anywhere
   import com.example.OldClass;  // Deleted last week
   ```

4. **Did I test edge cases?**
   - Empty lists
   - Null inputs  
   - Very large inputs
   - Special characters

---

## 6. Show Your Work (When It Matters)
**Complex logic needs inline rationale.**

For non-obvious decisions, add a comment explaining **why**, not what:

```python
# ❌ USELESS - explains the obvious
# Loop through users
for user in users:
    ...

# ✅ USEFUL - explains the reasoning
# Can't use batch updates due to MySQL 5.7 bug with CASE statements (MYSQL-12345)
# Processing individually prevents data corruption
for user in users:
    ...
```

### **When to comment:**
- Non-obvious algorithms or optimizations
- Workarounds for bugs in dependencies
- Business logic that seems weird but is correct
- Security-critical decisions

### **When NOT to comment:**
- What the code does (should be self-evident)
- Obvious variable names
- Standard library usage

---

## 7. Configuration Over Magic
**Make behavior controllable. Avoid hidden defaults.**

```python
# ❌ BRITTLE (hard-coded assumption)
def read_file(path):
    # Assumes UTF-8, will break on legacy Windows files
    return open(path).read()

# ✅ CONTROLLABLE
def read_file(path, encoding='utf-8'):
    return open(path, encoding=encoding).read()
```

### **Why this matters:**
Hard-coded assumptions become bugs when reality differs:
- UTF-8 assumption breaks on Windows-1252 files
- Timeout of 5 seconds fails on slow networks
- Hard-coded API keys get committed to git

### **Balance:**
Don't add 10 parameters "just in case." Add parameters when:
- The default will break for known legitimate use cases
- Testing requires different values
- You're replacing a hard-coded value that already caused a bug

---

## 8. Fail Fast and Loud
**Catch problems at the boundary, not deep in the stack.**

Validate inputs at function/method boundaries. Let the caller deal with bad data, don't let it corrupt internal state.

### **Example - What NOT to do:**
```java
public void processOrder(Order order) {
    // ❌ Fails deep in the stack with cryptic error
    // NullPointerException at line 47, but order was null at line 5
    validateOrder(order);  // Might not check for null
    saveToDatabase(order);
    sendConfirmation(order.getCustomerEmail());
}
```

### **Example - What TO do:**
```java
public void processOrder(Order order) {
    // ✅ Fail fast with clear message
    if (order == null) {
        throw new IllegalArgumentException("Order cannot be null");
    }
    if (order.getItems().isEmpty()) {
        throw new IllegalArgumentException("Order must contain at least one item");
    }
    if (order.getCustomerEmail() == null || !isValidEmail(order.getCustomerEmail())) {
        throw new IllegalArgumentException("Valid customer email is required");
    }
    
    // Now we know order is valid, proceed safely
    saveToDatabase(order);
    sendConfirmation(order.getCustomerEmail());
}
```

### **Principle:**
Make error messages self-explanatory. The exception should tell the user exactly what to do.

---

## Meta-Rule: Know When to Break the Rules

These guidelines aren't religious dogma:

- **Throwaway script?** Skip tests, name things tersely, combine methods.
- **Hotfix in production?** Surgical change trumps "beautiful code."
- **Performance-critical loop?** Optimize first, clarify with comments.
- **Working with legacy code?** Match existing style, even if it violates these rules.

### **But:**
When you break a rule, say so explicitly:
> "This violates 'Simplicity' by adding 3 parameters, but the API needs to support JSON, XML, and CSV formats per requirements. The complexity is necessary here."

---

## Autonomous Coding Protocol
**For when you're coding without real-time user feedback.**

When working autonomously, you must make decisions without asking. Use this protocol:

### Confidence Levels & Actions

| Confidence | Action | Examples |
|------------|--------|----------|
| **95-100%** | Proceed confidently | Adding CRUD endpoint following existing patterns, implementing standard validation |
| **80-94%** | Proceed with safe default | Tech choice: pick industry standard (PostgreSQL over exotic DB). Architecture: use existing project patterns |
| **60-79%** | Proceed with conservative choice + note | "Using REST instead of GraphQL (could be either, REST is safer default)" |
| **40-59%** | Conservative approach + detailed comment | "Assuming user wants email notifications - added flag to disable if not needed" |
| **<40%** | STOP - cannot proceed autonomously | Requirements completely unclear, multiple incompatible interpretations |

### Safe Defaults for Ambiguous Decisions

**When unsure, always choose:**

1. **Simplicity over complexity**
   - REST API > GraphQL (unless specified)
   - Monolith > Microservices (unless scale requires)
   - Postgres > MongoDB (unless document structure obvious)
   - Server components > Client components (Next.js)

2. **Existing patterns over new ones**
   - Match existing code style exactly
   - Use same libraries already in project
   - Follow established folder structure

3. **Conservative over aggressive**
   - Synchronous > Async (unless performance critical)
   - Server-side rendering > Client-side (for SEO/content)
   - Pessimistic locking > Optimistic (for critical data)

4. **Standard over custom**
   - JWT for auth (industry standard)
   - bcrypt for passwords
   - UUIDs for IDs (not sequential)
   - ISO 8601 for dates

### Red Lines: When to STOP Immediately

**Never proceed autonomously if:**

- 🛑 Requirements are contradictory ("must be both sync and async")
- 🛑 Security critical decision unclear (auth method, encryption)
- 🛑 Multiple incompatible architectural approaches seem valid
- 🛑 User intent genuinely ambiguous (not just needing a default)
- 🛑 Would require deleting/modifying significant existing code without clear justification
- 🛑 Touching production credentials, secrets, or infrastructure configs

**In these cases:**
1. Document your uncertainty clearly
2. State the options you see
3. Recommend your best guess with reasoning
4. Explicitly flag: "⚠️ REQUIRES USER DECISION - Proceeding with X but Y or Z may be better"

### Decision Documentation Template

For any non-obvious decision made autonomously, add a comment:

```
// DECISION: [What you decided]
// REASON: [Why you chose this]
// ALTERNATIVES: [What else you considered]
// CONFIDENCE: [High/Medium/Low]
// REVERSIBLE: [Yes/No - can it be changed later easily?]
```

### The "Senior Engineer" Test

Before finalizing autonomous work, imagine explaining to a senior engineer:

1. **Can I explain every architectural choice in 1 sentence?**
   - Bad: "I used a factory pattern with dependency injection and..."
   - Good: "Used repository pattern to separate data access from business logic"

2. **Would they agree with my trade-offs?**
   - "I chose simplicity over performance because the user didn't mention scale"

3. **Are there any "WTF" moments?**
   - If you'd have to explain why you did something weird, don't do it

### Emergency Stop Conditions

**Immediately pause and flag for help if:**

1. **Same error persists after 3 fix attempts**
   - You're going in circles
   - Each attempt is making it worse

2. **Code is growing but problem isn't solved**
   - 200 lines written, still doesn't compile
   - "Almost working" for the last hour

3. **You're doing something you don't fully understand**
   - Copy-pasting code you can't explain
   - Using "solutions" from StackOverflow without understanding

4. **Requirements keep expanding**
   - Started with "add login button"
   - Now implementing OAuth2 with 2FA and biometric auth

5. **Fundamental realization:**
   - "The whole approach is wrong"
   - "I need to delete everything and start over"

**In these cases:**
- STOP all coding immediately
- Summarize what you attempted
- State what you learned
- Ask for direction

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

---

## Available Custom Agents

This repository includes specialized agents for different tasks. Select the appropriate agent in Copilot Chat:

- **@java-backend** - Java + Spring Boot + PostgreSQL + MongoDB + Redis development
- **@nodejs-backend** - Node.js/TypeScript API development
- **@react-frontend** - React + TypeScript frontend development
- **@devops** - Docker, infrastructure, and deployment
- **@code-review** - Code review, refactoring, and security analysis
- **@architect** - Software architecture and design patterns
- **@debugger** - Debugging and troubleshooting
- **@doc-writer** - Documentation writing
- **@designer** - UI/UX design and theming

---

**Remember:** Your job is to solve the stated problem, not to anticipate every possible future need. When in doubt, do less.
