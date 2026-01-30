---
name: java-coding
description: Core Java coding guidelines with Lombok and modern language practices. Use for pure Java code without frameworks. For Spring Boot, use springboot-coding skill.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Java Coding Guidelines (Language Core)

**IMPORTANT:** This skill is for **core Java language** - no Spring, no Jakarta EE. For framework-specific code, see:
- `springboot-coding/` - Spring Boot applications
- `java-design-patterns/` - Design patterns implementation
- `logging/` - Logging practices

## Quick Start Checklist

Before writing Java code:
- [ ] Use Java 17+ features (records, switch expressions)
- [ ] Add Lombok to reduce boilerplate
- [ ] Never use fully qualified names inline
- [ ] Use Optional instead of null returns
- [ ] Validate inputs at method boundaries

---

## Core Dependencies (Always Include)

```xml
<!-- REQUIRED: Lombok for boilerplate reduction -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <scope>provided</scope>
</dependency>

<!-- REQUIRED: MapStruct for object mapping -->
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.5.Final</version>
</dependency>

<!-- OPTIONAL: Validation API (can use without Spring) -->
<dependency>
    <groupId>jakarta.validation</groupId>
    <artifactId>jakarta.validation-api</artifactId>
</dependency>
```

---

## Lombok Patterns (Use These)

### 1. DTOs and Value Objects

```java
import lombok.Data;
import lombok.Builder;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class UserDto {
    private String id;
    private String email;
    private String name;
}

// Usage
UserDto dto = UserDto.builder()
    .id(UUID.randomUUID().toString())
    .email("user@example.com")
    .name("John")
    .build();
```

**What each annotation does:**
- `@Data` - Generates getters, setters, toString, equals, hashCode
- `@Builder` - Creates fluent builder pattern
- `@NoArgsConstructor` - Empty constructor (needed for some frameworks)
- `@AllArgsConstructor` - Constructor with all fields

### 2. Domain Entities (DO NOT use @Data for JPA!)

```java
import lombok.Getter;
import lombok.Setter;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.EqualsAndHashCode;
import lombok.ToString;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@EqualsAndHashCode(of = "id")  // Only use id for equals/hashCode
@ToString(exclude = "password") // Never include password in toString
public class User {
    private UUID id;
    private String email;
    private String name;
    private String password;  // Will be excluded from toString
}
```

**CRITICAL:** Never use `@Data` on JPA entities because it generates `equals()` and `hashCode()` using ALL fields, which breaks lazy loading and causes performance issues.

### 3. Services

```java
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

@Slf4j
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;  // final = immutable
    private final UserMapper userMapper;
    
    public UserDto findById(UUID id) {
        log.info("Finding user by id: {}", id);
        return userRepository.findById(id)
            .map(userMapper::toDto)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
}
```

**Why `@RequiredArgsConstructor`:**
- Generates constructor with all `final` fields
- Enables dependency injection without `@Autowired`
- Makes dependencies explicit and immutable

---

## MapStruct (Object Mapping)

MapStruct generates mapping code at compile time (not runtime reflection).

```java
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.ReportingPolicy;

@Mapper(unmappedTargetPolicy = ReportingPolicy.ERROR)
public interface UserMapper {
    
    // Simple mapping (fields with same name auto-map)
    UserDto toDto(User user);
    
    // Reverse mapping
    User toEntity(UserDto dto);
    
    // Custom mappings with @Mapping
    @Mapping(target = "id", ignore = true)  // Don't map id
    @Mapping(target = "createdAt", ignore = true)  // Don't map audit field
    User toEntity(UserCreateDto dto);
}
```

**CRITICAL:** Add MapStruct annotation processor to your build:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok</artifactId>
                        <version>1.18.30</version>
                    </path>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>1.5.5.Final</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

---

## Package Structure

```
src/main/java/com/company/project/
├── domain/                    # Business logic
│   ├── user/
│   │   ├── User.java          # Entity
│   │   ├── UserDto.java       # DTO
│   │   ├── UserRepository.java # Interface (not implementation)
│   │   ├── UserService.java   # Business logic
│   │   └── UserMapper.java    # Object mapping
│   └── order/
│       └── ...
├── infrastructure/            # Technical implementations
│   ├── persistence/           # Repository implementations
│   └── web/                   # Controllers (if not using Spring)
└── shared/                    # Cross-cutting concerns
    ├── exception/
    └── util/
```

---

## Naming Conventions

| Type | Pattern | Example | Why |
|------|---------|---------|-----|
| Class | PascalCase | `UserService` | Java convention |
| Interface | PascalCase | `UserRepository` | Same as class |
| Method | camelCase verb-first | `findById()` | Clear intent |
| Variable | camelCase | `userEmail` | Readability |
| Constant | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` | Distinguish from variables |
| Generic | Single uppercase | `T`, `E`, `K`, `V` | Java convention |

---

## Exception Handling

### Business Exception Hierarchy

```java
import lombok.Getter;

@Getter
public abstract class BusinessException extends RuntimeException {
    private final String errorCode;
    private final int httpStatus;
    
    protected BusinessException(String errorCode, int httpStatus, String message) {
        super(message);
        this.errorCode = errorCode;
        this.httpStatus = httpStatus;
    }
}

// Specific exceptions
public class UserNotFoundException extends BusinessException {
    public UserNotFoundException(UUID id) {
        super("USER_NOT_FOUND", 404, "User with id " + id + " not found");
    }
}

public class DuplicateEmailException extends BusinessException {
    public DuplicateEmailException(String email) {
        super("DUPLICATE_EMAIL", 409, "Email already registered: " + email);
    }
}

public class ValidationException extends BusinessException {
    public ValidationException(String message) {
        super("VALIDATION_ERROR", 400, message);
    }
}
```

### Using Exceptions

```java
public UserDto findById(UUID id) {
    // ✅ Use Optional with orElseThrow
    return userRepository.findById(id)
        .map(mapper::toDto)
        .orElseThrow(() -> new UserNotFoundException(id));
}

public UserDto create(UserCreateDto dto) {
    // ✅ Validate before creating
    if (userRepository.existsByEmail(dto.getEmail())) {
        throw new DuplicateEmailException(dto.getEmail());
    }
    // ... create user
}

// ❌ WRONG: Returning null
public UserDto findByEmail(String email) {
    User user = userRepository.findByEmail(email);
    if (user == null) {
        return null;  // NEVER do this!
    }
    return mapper.toDto(user);
}

// ✅ CORRECT: Return Optional
public Optional<UserDto> findByEmail(String email) {
    return userRepository.findByEmail(email)
        .map(mapper::toDto);
}
```

---

## Validation (Jakarta Bean Validation)

### DTO Validation

```java
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.Data;
import lombok.Builder;

@Data
@Builder
public class UserCreateDto {
    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    private String email;
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be 2-100 characters")
    private String name;
    
    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;
}
```

### Manual Validation (without Spring)

```java
import jakarta.validation.Validation;
import jakarta.validation.Validator;
import jakarta.validation.ValidatorFactory;
import jakarta.validation.ConstraintViolation;

public class ValidationUtil {
    private static final Validator validator;
    
    static {
        try (ValidatorFactory factory = Validation.buildDefaultValidatorFactory()) {
            validator = factory.getValidator();
        }
    }
    
    public static <T> void validate(T object) {
        Set<ConstraintViolation<T>> violations = validator.validate(object);
        if (!violations.isEmpty()) {
            String message = violations.stream()
                .map(v -> v.getPropertyPath() + ": " + v.getMessage())
                .collect(Collectors.joining(", "));
            throw new ValidationException(message);
        }
    }
}

// Usage
UserCreateDto dto = UserCreateDto.builder()
    .email("invalid-email")
    .build();

ValidationUtil.validate(dto);  // Throws ValidationException
```

---

## Import Conventions (CRITICAL)

**ALWAYS use imports - NEVER use fully qualified names inline:**

```java
// ❌ WRONG - Never do this
java.util.List<String> items = new java.util.ArrayList<>();
com.example.project.User user = new com.example.project.User();

// ✅ CORRECT - Always use imports
import java.util.List;
import java.util.ArrayList;
import java.util.UUID;
import java.util.Optional;
import java.time.Instant;

import com.example.project.User;

List<String> items = new ArrayList<>();
User user = new User();
```

### Import Organization (IDE should handle this)

```java
// Group 1: java.* and javax.* (alphabetically)
import java.time.Instant;
import java.util.Optional;
import java.util.UUID;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;

// Group 2: External libraries (alphabetically)
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import org.mapstruct.Mapper;
import org.mapstruct.Mapping;

// Group 3: Internal project imports
import com.company.project.shared.exception.BusinessException;
```

---

## Modern Java Features (Java 17+)

### 1. Records for DTOs (Immutable Data)

```java
// Immutable data carrier - perfect for DTOs
public record UserDto(UUID id, String email, String name) {}

public record UserCreateDto(
    @Email @NotBlank String email,
    @NotBlank @Size(min = 2) String name
) {}

// Usage
UserDto dto = new UserDto(id, email, name);
String email = dto.email();  // Getter without "get" prefix
```

**When to use Records:**
- ✅ DTOs that just carry data
- ✅ Value objects
- ✅ Immutable data

**When NOT to use Records:**
- ❌ JPA entities (need setters, lazy loading)
- ❌ Classes with business logic
- ❌ Classes that need inheritance

### 2. Optional (Never Return Null)

```java
// ✅ Return Optional for methods that might not find results
public Optional<User> findByEmail(String email) {
    return users.stream()
        .filter(u -> u.getEmail().equals(email))
        .findFirst();
}

// ✅ Use orElseThrow with custom exceptions
User user = findByEmail(email)
    .orElseThrow(() -> new UserNotFoundException(email));

// ✅ Provide default with orElse/orElseGet
String name = findById(id)
    .map(User::getName)
    .orElse("Unknown");

// ✅ Chain operations
String upperEmail = findById(id)
    .map(User::getEmail)
    .map(String::toUpperCase)
    .orElse("NO EMAIL");

// ❌ WRONG - Never return null
public User findById(UUID id) {
    // ... returns null if not found
}

// ❌ WRONG - Never use Optional.get() without checking
User user = findById(id).get();  // Can throw NoSuchElementException
```

### 3. Switch Expressions (Java 14+)

```java
// Old way (verbose)
public String getStatusDescription(Status status) {
    switch (status) {
        case PENDING:
            return "Awaiting processing";
        case PROCESSING:
            return "Currently processing";
        case COMPLETED:
            return "Finished successfully";
        default:
            return "Unknown";
    }
}

// ✅ Modern way - Switch expression
public String getStatusDescription(Status status) {
    return switch (status) {
        case PENDING -> "Awaiting processing";
        case PROCESSING -> "Currently processing";
        case COMPLETED -> "Finished successfully";
        case FAILED -> "Processing failed";
    };  // No default needed if all cases covered
}

// With yield for complex logic
public int calculatePriority(Order order) {
    return switch (order.getType()) {
        case STANDARD -> 1;
        case EXPRESS -> 5;
        case URGENT -> {
            log.info("Urgent order detected");
            yield 10;  // Use yield when you need multiple statements
        }
    };
}
```

### 4. Text Blocks (Java 15+)

```java
// Old way (ugly with \n and +)
String json = "{\n" +
    "  \"name\": \"" + user.getName() + "\",\n" +
    "  \"email\": \"" + user.getEmail() + "\"\n" +
    "}";

// ✅ Modern way - Text block
String json = """
    {
        "name": "%s",
        "email": "%s",
        "createdAt": "%s"
    }
    """.formatted(user.getName(), user.getEmail(), Instant.now());

// SQL queries
String query = """
    SELECT u.id, u.email, u.name
    FROM users u
    WHERE u.active = true
    AND u.created_at > ?
    ORDER BY u.created_at DESC
    """;
```

### 5. Pattern Matching (Java 17 Preview, 21 Standard)

```java
// instanceof pattern matching
if (obj instanceof User user) {
    // user is automatically cast to User type
    System.out.println(user.getEmail());
    // No need for: User user = (User) obj;
}

// switch pattern matching (Java 21)
public String describe(Object obj) {
    return switch (obj) {
        case User u -> "User: " + u.getEmail();
        case Order o -> "Order: " + o.getId();
        case null -> "null";
        default -> "Unknown type";
    };
}
```

### 6. var for Local Variables (Java 10+)

```java
// ✅ Use var when type is obvious from context
var user = new User();  // Type is User
var users = new ArrayList<User>();  // Type is ArrayList<User>
var name = "John";  // Type is String

// ❌ Don't use var when type is not clear
var result = someMethod();  // What type is result?

// ✅ Better
UserDto result = someMethod();
```

---

## Common Mistakes and How to Avoid Them

### Mistake 1: NullPointerException

```java
// ❌ WRONG - No null checking
String name = user.getName().toUpperCase();  // NPE if user or name is null

// ✅ CORRECT - Defensive programming
String name = Optional.ofNullable(user)
    .map(User::getName)
    .map(String::toUpperCase)
    .orElse("UNKNOWN");
```

### Mistake 2: Modifying collections while iterating

```java
// ❌ WRONG - ConcurrentModificationException
List<User> users = getUsers();
for (User user : users) {
    if (user.isInactive()) {
        users.remove(user);  // Throws exception!
    }
}

// ✅ CORRECT - Use Iterator or removeIf
users.removeIf(User::isInactive);

// Or use Iterator explicitly
Iterator<User> iterator = users.iterator();
while (iterator.hasNext()) {
    if (iterator.next().isInactive()) {
        iterator.remove();
    }
}
```

### Mistake 3: String concatenation in loops

```java
// ❌ WRONG - O(n²) performance
String result = "";
for (String s : strings) {
    result += s + ", ";  // Creates new String object each iteration
}

// ✅ CORRECT - Use StringBuilder
StringBuilder sb = new StringBuilder();
for (String s : strings) {
    sb.append(s).append(", ");
}
String result = sb.toString();

// Or better, use String.join
String result = String.join(", ", strings);
```

### Mistake 4: Not closing resources

```java
// ❌ WRONG - Resource leak
InputStream is = new FileInputStream("file.txt");
// If exception thrown here, stream never closes

// ✅ CORRECT - try-with-resources
try (InputStream is = new FileInputStream("file.txt")) {
    // Use stream
}  // Automatically closed, even if exception occurs

// Multiple resources
try (var in = new FileInputStream("in.txt");
     var out = new FileOutputStream("out.txt")) {
    // Copy data
}
```

### Mistake 5: Using == for String comparison

```java
// ❌ WRONG - Compares object references, not content
if (user.getEmail() == "admin@example.com") { ... }

// ✅ CORRECT - Use equals()
if ("admin@example.com".equals(user.getEmail())) { ... }

// Even better (handles null safely)
if (Objects.equals(user.getEmail(), "admin@example.com")) { ... }
```

---

## Best Practices Summary

### Always Do
- ✅ Use `final` for fields and variables that don't change
- ✅ Use `var` for local variables when type is obvious
- ✅ Use `Optional` instead of null returns
- ✅ Use `Stream` API for collection operations
- ✅ Validate inputs at method boundaries
- ✅ Use try-with-resources for AutoCloseable objects
- ✅ Use records for immutable DTOs
- ✅ Use switch expressions over traditional switch
- ✅ Use text blocks for multi-line strings
- ✅ Add @ToString(exclude) for sensitive fields

### Never Do
- ❌ Return null from methods (use Optional instead)
- ❌ Use == for String comparison (use equals())
- ❌ Concatenate strings in loops (use StringBuilder)
- ❌ Leave resources open (always close streams, connections)
- ❌ Use fully qualified names inline (always import)
- ❌ Use @Data on JPA entities
- ❌ Catch generic Exception without rethrowing or logging
- ❌ Ignore compiler warnings (fix them)
- ❌ Use raw types (List instead of List<String>)
- ❌ Ignore equals() and hashCode() for entity classes

---

## Testing Helpers

### Test Data Builders

```java
public class UserTestBuilder {
    public static User.UserBuilder aUser() {
        return User.builder()
            .id(UUID.randomUUID())
            .email("test@example.com")
            .name("Test User")
            .active(true);
    }
    
    public static User.UserBuilder anAdmin() {
        return aUser().email("admin@example.com");
    }
    
    public static User.UserBuilder anInactiveUser() {
        return aUser().active(false);
    }
}

// Usage in tests
User user = UserTestBuilder.aUser()
    .email("specific@example.com")
    .build();

User admin = UserTestBuilder.anAdmin().build();
```

This pattern makes tests readable and reduces duplication.

---

## Common AI Coding Mistakes (Autonomous Mode)

**When coding without user feedback, avoid these AI-specific errors:**

### 1. Hallucinating APIs

**The Mistake:** Using methods or classes that don't exist
```java
// ❌ WRONG - Method doesn't exist
user.setActive(true);  // Lombok @Getter doesn't generate setters for individual fields
list.filter(predicate);  // List doesn't have filter() - use stream()
map.getOrDefault(key, defaultValue, mapper);  // No 3-parameter version
```

**How to avoid:**
- Check actual class documentation
- Use IDE autocomplete features
- Verify method signatures exist
- When unsure, use standard JDK methods (stream API, Optional, etc.)

### 2. Forgetting Null Checks in Chains

**The Mistake:** Chaining calls without null checks
```java
// ❌ WRONG - Will NPE if any part is null
String city = user.getAddress().getCity().toUpperCase();

// ✅ CORRECT - Safe chaining with Optional
String city = Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .map(String::toUpperCase)
    .orElse("UNKNOWN");
```

**Rule:** Any chain with 3+ calls needs null safety.

### 3. Over-Engineering with Patterns

**The Mistake:** Using complex patterns for simple problems
```java
// ❌ WRONG - Factory + Strategy + Builder for simple discount
public interface DiscountStrategy { }
public class DiscountFactory { }
public class DiscountBuilder { }
// 50 lines of code...

// ✅ CORRECT - Simple method
public BigDecimal calculateDiscount(Order order) {
    return order.getTotal().compareTo(new BigDecimal("100")) >= 0
        ? order.getTotal().multiply(new BigDecimal("0.10"))
        : BigDecimal.ZERO;
}
```

**Rule:** If it fits in 20 lines, don't use a pattern.

### 4. Missing Import Statements

**The Mistake:** Using classes without importing
```java
// ❌ WRONG - Missing imports
public void process(List<String> items) {
    UUID id = randomUUID();  // Missing: import java.util.UUID
    items.stream().collect(toList());  // Missing: import static
}
```

**Rule:** Every non-java.lang class needs import.

### 5. Confusing Similar Methods

**The Mistake:** Using wrong method for the context
```java
// ❌ WRONG - collect(Collectors.toList()) on immutable list
List<String> result = immutableList.stream()
    .collect(Collectors.toList());  // Works but unnecessary

// ✅ CORRECT - Use List.copyOf for immutable
List<String> result = List.copyOf(immutableList);

// ❌ WRONG - isEmpty() vs == null
if (list.isEmpty()) { }  // NPE if list is null

// ✅ CORRECT - Check null first
if (list == null || list.isEmpty()) { }
```

### 6. Forgetting equals() and hashCode()

**The Mistake:** Not implementing equals/hashCode for entities
```java
// ❌ WRONG - Lombok @Data on entity (includes equals/hashCode)
@Data
@Entity
public class User {
    private Long id;  // equals() compares all fields including collections!
    private List<Order> orders;  // This breaks lazy loading
}

// ✅ CORRECT - Use @Getter/@Setter, exclude collections from equals
@Entity
@Getter @Setter
@EqualsAndHashCode(of = "id")  // Only compare IDs
public class User {
    @Id
    private Long id;
    
    @OneToMany
    @EqualsAndHashCode.Exclude  // Exclude from equals
    private List<Order> orders;
}
```

### 7. Autonomous Decision Checklist

**Before committing code, verify:**

- [ ] All imports are present (no red squiggles)
- [ ] No method calls on potentially null objects
- [ ] All used methods actually exist in the classes
- [ ] No complex patterns for simple logic
- [ ] equals() and hashCode() properly configured for entities
- [ ] No fully qualified names inline (use imports)
- [ ] All compiler warnings addressed
- [ ] No raw types (List<String> not List)

**If you can't verify:** Add a comment explaining your uncertainty:
```java
// UNCERTAIN: Not sure if UserRepository has this method
// ASSUMPTION: Assuming it follows standard Spring Data naming
// REVIEW NEEDED: Please verify this query works as expected
List<User> findByActiveTrueAndCreatedAfter(LocalDate date);
```
