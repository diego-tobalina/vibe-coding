---
description: Universal coding style rules for naming, structure, and error handling
globs:
  - "**/*.java"
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
  - "**/*.py"
---

# Coding Style Rules

**CRITICAL:** These rules are non-negotiable. Follow them consistently across all code.

---

## Autonomous Decision Making (For Solo Coding)

When coding without user feedback, use this confidence framework:

### Confidence Levels

| Level | Confidence | Action | Examples |
|-------|-----------|--------|----------|
| **1** | 95-100% | ✅ Proceed confidently | Adding standard CRUD endpoint, simple validation, following existing patterns |
| **2** | 80-94% | ✅ Proceed with safe default | Tech choice (PostgreSQL vs MongoDB), naming conventions, simple refactoring |
| **3** | 60-79% | ⚠️ Proceed with comment | "Using REST instead of GraphQL (REST is safer default)", "Assuming UTC timezone" |
| **4** | 40-59% | ⚠️ Conservative + detailed comment | Add `// UNCERTAIN:` comment explaining decision and alternatives |
| **5** | <40% | 🛑 STOP - Cannot proceed | Requirements contradictory, security-critical unclear, multiple valid approaches |

### When to Add Uncertainty Comments

Add a comment when confidence is 40-79%:

```java
// UNCERTAIN: Multiple approaches possible here
// DECISION: Using Strategy pattern for discount calculation
// ALTERNATIVES: Could use simple if/else (less flexible) or rules engine (overkill)
// REASONING: Strategy allows adding new discount types without modifying existing code
// CONFIDENCE: 65%
// REVIEW: Verify this matches team's preference for design patterns
```

### Safe Defaults (When Unsure)

Always default to:

1. **Simpler over complex** (20-line method beats 5-class pattern)
2. **Standard over custom** (REST > GraphQL unless specified)
3. **Conservative over aggressive** (sync > async unless required)
4. **Existing patterns** (match current codebase exactly)

### Red Lines (Never Proceed Alone)

🛑 **STOP and flag for review if:**
- Security decisions (auth method, encryption, PII handling)
- Database schema changes
- API versioning decisions
- Breaking changes to existing functionality
- Production configuration changes
- Performance optimizations (measure first)

---

## Naming Conventions (INFLEXIBLE)

| Type | Convention | Example | ❌ WRONG | ✅ CORRECT |
|------|------------|---------|----------|------------|
| Variable | camelCase | userEmail | user_email, UserEmail | userEmail |
| Constant | UPPER_SNAKE_CASE | MAX_RETRY_COUNT | maxRetryCount, MaxRetryCount | MAX_RETRY_COUNT |
| Function | camelCase (verb) | calculateTotal() | calc_total(), CalculateTotal() | calculateTotal() |
| Class | PascalCase | UserService | userService, user_service | UserService |
| Interface | PascalCase | PaymentProcessor | paymentProcessor, IPaymentProcessor | PaymentProcessor |
| File | Match class name | UserService.java | userService.java, service.java | UserService.java |
| Boolean | is/has/can prefix | isValid, hasPermission | valid, permission | isValid, hasPermission |
| Collection | Plural noun | users, orderItems | userList, orderItemArray | users, orderItems |
| Private | Leading underscore (JS only) | _internalMethod | internalMethod_ | _internalMethod |

### Naming Anti-Patterns (NEVER DO)

```java
// ❌ WRONG - Abbreviations
int cnt;           // count
String usr;        // user
List<Order> ords;  // orders

// ✅ CORRECT - Full words
int count;
String user;
List<Order> orders;

// ❌ WRONG - Meaningless names
void process() { }
int data = 10;
Object thing = new Object();

// ✅ CORRECT - Descriptive names
void processPayment() { }
int retryCount = 10;
Object userCache = new Object();

// ❌ WRONG - Similar names
User user1;
User user2;
User user3;

// ✅ CORRECT - Distinguishable names
User loggedInUser;
User targetUser;
User adminUser;
```

---

## Code Structure Rules

### Maximum Sizes (Hard Limits)

| Component | Max Size | When to Split |
|-----------|----------|---------------|
| Function/Method | 20 lines | More than 1 responsibility |
| Class | 400 lines | More than 1 concept |
| File | 500 lines | Multiple classes |
| Nesting Level | 4 levels | Deeper nesting |
| Parameters | 4 params | Use builder/DTO |
| Imports | 50 imports | Split file |

### Indentation & Spacing (INFLEXIBLE)

```java
// Indentation: 4 spaces for Java, 2 for JavaScript/TypeScript
public class Example {
    public void method() {  // 4 spaces
        if (condition) {    // 4 more spaces
            doSomething();  // 8 spaces
        }
    }
}

// Always use braces, even for single line
// ❌ WRONG
if (condition)
    doSomething();

// ✅ CORRECT
if (condition) {
    doSomething();
}

// Blank lines: Between logical sections
public void processOrder(Order order) {
    validateOrder(order);       // Validation section
    
    calculateTotal(order);      // Calculation section
    
    saveOrder(order);           // Persistence section
    
    sendConfirmation(order);    // Notification section
}
```

---

## Immutability Rules

### Always Prefer Immutable

```java
// ❌ WRONG - Mutable
public class User {
    private String name;
    
    public void setName(String name) {  // Mutable!
        this.name = name;
    }
}

// ✅ CORRECT - Immutable
public class User {
    private final String name;
    
    public User(String name) {
        this.name = name;
    }
    
    public User withName(String newName) {  // Returns new instance
        return new User(newName);
    }
}

// ❌ WRONG - Mutable collections
List<String> names = new ArrayList<>();
names.add("John");

// ✅ CORRECT - Immutable collections
List<String> names = List.of("John", "Jane");  // Java 9+
// Or
List<String> names = Collections.unmodifiableList(list);
```

---

## Error Handling Rules

### Fail Fast with Clear Messages

```java
// ❌ WRONG - Silent failure
public void processOrder(Order order) {
    if (order == null) {
        return;  // Silent failure!
    }
    // ...
}

// ✅ CORRECT - Fail fast with context
public void processOrder(Order order) {
    if (order == null) {
        throw new IllegalArgumentException("Order cannot be null");
    }
    if (order.getItems().isEmpty()) {
        throw new IllegalArgumentException(
            "Order must contain at least one item, orderId=" + order.getId()
        );
    }
    // ...
}
```

### Exception Rules

1. **Never catch generic Exception without rethrowing**
```java
// ❌ WRONG
try {
    processOrder(order);
} catch (Exception e) {
    log.error("Error");  // Swallows exception!
}

// ✅ CORRECT
// Option 1: Let it propagate
processOrder(order);

// Option 2: Catch specific, wrap, and throw
try {
    processOrder(order);
} catch (ValidationException e) {
    throw new OrderProcessingException("Invalid order", e);
}

// Option 3: Catch, log, and rethrow
try {
    processOrder(order);
} catch (Exception e) {
    log.error("Failed to process order: {}", order.getId(), e);
    throw e;  // Rethrow!
}
```

2. **Never swallow InterruptedException**
```java
// ❌ WRONG - Loses interrupt status
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    // Empty catch - WRONG!
}

// ✅ CORRECT - Restore interrupt status
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
    throw new RuntimeException("Thread interrupted", e);
}
```

---

## Comments Rules

### Comment What & Why, Not What

```java
// ❌ WRONG - Explains the obvious
// Increment counter
counter++;

// ❌ WRONG - Out of date comment
// Process orders (this actually processes payments now)
processPayments();

// ✅ CORRECT - Explains why
// We retry 3 times because the payment gateway occasionally times out
// but succeeds on retry. See ticket PAY-1234.
int maxRetries = 3;

// ✅ CORRECT - Documents workaround
// TODO: Remove this workaround after upgrading to Library v3.0
// which fixes the null pointer issue. See issue #456.
if (obj != null) {
    process(obj);
}
```

### Never Leave Commented-Out Code

```java
// ❌ WRONG - Commented code
public void process() {
    validate();
    // oldProcess();
    newProcess();
    // saveToDisk();
    saveToCloud();
}

// ✅ CORRECT - Clean code
public void process() {
    validate();
    newProcess();
    saveToCloud();
}

// If you need to see old code: use version control (git)
```

---

## Code Formatting Rules

### Line Length & Breaking

```java
// Maximum line length: 120 characters

// ❌ WRONG - Too long
public void process(Order order, Customer customer, Payment payment, ShippingAddress address, BillingAddress billing) {
    // ...
}

// ✅ CORRECT - Break parameters
public void process(
    Order order,
    Customer customer, 
    Payment payment,
    ShippingAddress address,
    BillingAddress billing
) {
    // ...
}

// ❌ WRONG - Long chain on one line
user.getOrders().stream().filter(o -> o.isActive()).map(o -> o.getTotal()).reduce(BigDecimal.ZERO, BigDecimal::add);

// ✅ CORRECT - Break chain
user.getOrders()
    .stream()
    .filter(Order::isActive)
    .map(Order::getTotal)
    .reduce(BigDecimal.ZERO, BigDecimal::add);
```

### Whitespace Rules

```java
// Space after keywords
if (condition) { }      // ✅
if(condition) { }       // ❌

for (int i = 0; i < n; i++) { }  // ✅
for(int i=0;i<n;i++) { }         // ❌

// Space around operators
int sum = a + b;        // ✅
int sum=a+b;            // ❌

// No space before semicolon
return value;           // ✅
return value ;          // ❌

// Space after commas
method(a, b, c);        // ✅
method(a,b,c);          // ❌
```

---

## Import Rules

### Organization (Alphabetical Groups)

```java
// Group 1: Java/Jakarta standard library (alphabetical)
import java.time.Instant;
import java.util.List;
import java.util.Optional;
import java.util.UUID;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;

// Group 2: Third-party libraries (alphabetical)
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;

import org.mapstruct.Mapper;
import org.springframework.stereotype.Service;

// Group 3: Project imports (alphabetical)
import com.company.project.domain.user.User;
import com.company.project.shared.exception.BusinessException;

// Static imports last
import static org.assertj.core.api.Assertions.assertThat;
```

### Never Use Wildcard Imports

```java
// ❌ WRONG
import java.util.*;
import lombok.*;

// ✅ CORRECT
import java.util.List;
import java.util.ArrayList;
import java.util.Optional;
import lombok.Data;
import lombok.Builder;
```

---

## Documentation Rules

### Java (Javadoc Required for Public APIs)

```java
/**
 * Calculates the total price for an order including taxes and discounts.
 * 
 * <p>The calculation applies the following rules:
 * <ul>
 *   <li>Subtotal: Sum of all item prices</li>
 *   <li>Tax: 8.25% of subtotal (calculated per item)</li>
 *   <li>Discount: Applied if discount code is valid</li>
 * </ul>
 *
 * @param order the order to calculate price for, must not be null
 * @param taxRate the tax rate as decimal (e.g., 0.0825 for 8.25%)
 * @param discountCode optional discount code, null if no discount
 * @return the calculated total price, never null
 * @throws IllegalArgumentException if order is null or tax rate is negative
 * @throws InvalidDiscountException if the discount code is invalid or expired
 * 
 * @since 1.2.0
 * @see Order
 * @see DiscountService#validateDiscount
 * 
 * @deprecated Use {@link #calculateTotalV2} instead, which supports multiple discounts
 */
public BigDecimal calculateTotal(Order order, BigDecimal taxRate, String discountCode) {
    // implementation
}
```

### TypeScript/JavaScript (JSDoc for Public Functions)

```typescript
/**
 * Fetches user data from the API with caching.
 * 
 * This function implements a cache-aside pattern where we first check
 * the Redis cache, then fall back to the database if not cached.
 * Results are cached for 5 minutes (TTL = 300 seconds).
 * 
 * @param userId - The unique identifier of the user (UUID format)
 * @param options - Optional request configuration
 * @param options.includeProfile - Whether to include extended profile data (default: false)
 * @param options.forceRefresh - Whether to bypass cache and fetch fresh data (default: false)
 * @returns Promise resolving to user data, never null
 * @throws {NotFoundError} When user is not found
 * @throws {AuthenticationError} When the request lacks valid authentication
 * @throws {RateLimitError} When API rate limit is exceeded
 * 
 * @example
 * // Basic usage
 * const user = await fetchUser('550e8400-e29b-41d4-a716-446655440000');
 * console.log(user.name);
 * 
 * @example
 * // With options
 * const user = await fetchUser('550e8400-e29b-41d4-a716-446655440000', {
 *   includeProfile: true,
 *   forceRefresh: true
 * });
 * 
 * @since 2.1.0
 */
async function fetchUser(
  userId: string, 
  options?: { includeProfile?: boolean; forceRefresh?: boolean }
): Promise<User> {
  // implementation
}
```

---

## Enforcement

### IDE Configuration

**IntelliJ IDEA:**
```xml
<!-- .idea/codeStyles/Project.xml -->
<code_scheme name="Project">
  <JavaCodeStyleSettings>
    <option name="CLASS_COUNT_TO_USE_IMPORT_ON_DEMAND" value="999" />
    <option name="NAMES_COUNT_TO_USE_IMPORT_ON_DEMAND" value="999" />
  </JavaCodeStyleSettings>
  <codeStyleSettings language="JAVA">
    <option name="RIGHT_MARGIN" value="120" />
    <option name="METHOD_PARAMETERS_WRAP" value="5" />
    <option name="METHOD_PARAMETERS_LPAREN_ON_NEXT_LINE" value="true" />
    <option name="METHOD_PARAMETERS_RPAREN_ON_NEXT_LINE" value="true" />
  </codeStyleSettings>
</code_scheme>
```

**VS Code:**
```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.rulers": [120],
  "editor.tabSize": 2,
  "typescript.format.insertSpaceAfterOpeningAndBeforeClosingNonemptyBraces": true,
  "java.format.settings.url": "https://raw.githubusercontent.com/google/styleguide/gh-pages/eclipse-java-google-style.xml"
}
```

### Linting Tools

- **Java:** Checkstyle, Spotless
- **TypeScript:** ESLint with @typescript-eslint
- **JavaScript:** ESLint, Prettier
- **Python:** Black, isort, flake8

---

## Violation Consequences

**Level 1 - Formatting:** Use IDE auto-format (Ctrl+Alt+L / Cmd+Option+L)
**Level 2 - Naming:** Refactor immediately, breaks code review
**Level 3 - Structure:** Must be fixed before merge
**Level 4 - Error Handling:** Blocks deployment

Remember: Consistent code style reduces cognitive load and bugs.
