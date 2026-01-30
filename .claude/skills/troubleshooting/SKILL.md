---
name: troubleshooting
description: Quick solutions for common development errors across Java, Node.js, React, and Spring Boot. Use when encountering errors you don't recognize or need fast solutions.
disable-model-invocation: true
allowed-tools:
  - Read
  - Grep
  - Bash
---

# Troubleshooting Guide

Quick solutions for common errors. Find your error message and follow the fix.

---

## How to Use This Guide

1. **Find your error** - Look for the exact or similar error message
2. **Check the cause** - Understand why it's happening
3. **Apply the fix** - Follow the solution steps
4. **Verify** - Confirm the error is resolved

---

## Java Errors

### NullPointerException (NPE)

**Error Message:**
```
java.lang.NullPointerException: Cannot invoke "String.length()" because "user.getName()" is null
```

**Cause:** Trying to use an object that is null

**Quick Fix:**
```java
// ❌ WRONG - Direct access
String name = user.getName().toUpperCase();

// ✅ CORRECT - Null check
String name = user != null && user.getName() != null 
    ? user.getName().toUpperCase() 
    : "UNKNOWN";

// ✅ BETTER - Use Optional
String name = Optional.ofNullable(user)
    .map(User::getName)
    .map(String::toUpperCase)
    .orElse("UNKNOWN");
```

**Prevention:**
- Use Optional for methods that might return null
- Validate inputs at method boundaries
- Use @NonNull annotations

---

### ClassNotFoundException / NoClassDefFoundError

**Error Message:**
```
java.lang.ClassNotFoundException: com.example.UserService
java.lang.NoClassDefFoundError: Could not initialize class com.example.Config
```

**Causes:**
- Missing dependency in pom.xml/build.gradle
- Classpath issues
- Dependency version conflicts

**Quick Fix:**
```xml
<!-- Add missing dependency -->
<dependency>
    <groupId>com.example</groupId>
    <artifactId>my-library</artifactId>
    <version>1.0.0</version>
</dependency>

<!-- Or fix version conflict -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>6.0.0</version>
</dependency>
```

**Check:**
```bash
# Maven
mvn dependency:tree | grep spring-core

# Gradle
./gradlew dependencies --configuration runtimeClasspath
```

---

### ConcurrentModificationException

**Error Message:**
```
java.util.ConcurrentModificationException
    at java.base/java.util.ArrayList$Itr.checkForComodification(ArrayList.java:1013)
```

**Cause:** Modifying collection while iterating

**Quick Fix:**
```java
// ❌ WRONG
for (User user : users) {
    if (user.isInactive()) {
        users.remove(user);  // Throws exception!
    }
}

// ✅ CORRECT - Use Iterator
Iterator<User> iterator = users.iterator();
while (iterator.hasNext()) {
    if (iterator.next().isInactive()) {
        iterator.remove();
    }
}

// ✅ CORRECT - Use removeIf (Java 8+)
users.removeIf(User::isInactive);

// ✅ CORRECT - Create new collection
List<User> activeUsers = users.stream()
    .filter(u -> !u.isInactive())
    .collect(Collectors.toList());
```

---

### OutOfMemoryError (OOM)

**Error Message:**
```
java.lang.OutOfMemoryError: Java heap space
java.lang.OutOfMemoryError: GC overhead limit exceeded
```

**Quick Fixes:**

1. **Increase heap size:**
```bash
java -Xmx2g -Xms2g -jar application.jar
```

2. **Find memory leak:**
```bash
# Generate heap dump on OOM
java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/heapdump.hprof -jar app.jar

# Analyze with VisualVM or Eclipse MAT
```

3. **Common causes:**
- Loading large collections into memory
- Not closing streams/resources
- Static collections growing unbounded
- Memory leaks in caches

---

### LazyInitializationException (Spring/Hibernate)

**Error Message:**
```
org.hibernate.LazyInitializationException: failed to lazily initialize a collection of role: com.example.User.orders, could not initialize proxy - no Session
```

**Cause:** Accessing lazy-loaded collection after Hibernate session closed

**Quick Fix:**
```java
// Solution 1: Use JOIN FETCH in query
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders WHERE u.id = :id")
Optional<User> findByIdWithOrders(@Param("id") UUID id);

// Solution 2: Use @Transactional to keep session open
@Transactional(readOnly = true)
public UserDto getUser(UUID id) {
    User user = userRepository.findById(id).orElseThrow();
    return mapper.toDto(user);  // Access orders here - session still open
}

// Solution 3: Use EntityGraph
@EntityGraph(attributePaths = "orders")
@Query("SELECT u FROM User u WHERE u.id = :id")
Optional<User> findByIdWithOrders(@Param("id") UUID id);
```

---

## Spring Boot Errors

### Application Failed to Start

**Error Message:**
```
***************************
APPLICATION FAILED TO START
***************************

Description:
Parameter 0 of constructor in com.example.UserService required a bean of type 'com.example.UserRepository' that could not be found.
```

**Quick Fixes:**

1. **Missing @Repository:**
```java
@Repository  // Add this!
public interface UserRepository extends JpaRepository<User, UUID> {
}
```

2. **Component scanning issue:**
```java
@SpringBootApplication(scanBasePackages = "com.example")
public class Application {
}
```

3. **Missing dependency:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

---

### Port Already in Use

**Error Message:**
```
Web server failed to start. Port 8080 was already in use.
```

**Quick Fix:**
```bash
# Find and kill process
lsof -ti:8080 | xargs kill -9

# Or change port
server.port=8081
# Or
java -jar app.jar --server.port=8081
```

---

### Whitelabel Error Page

**Error Message:**
```
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.
```

**Cause:** Error occurred but no custom error handler

**Quick Fix:**
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handle(Exception e) {
        return ResponseEntity.status(500)
            .body(new ErrorResponse("INTERNAL_ERROR", e.getMessage()));
    }
}
```

---

## Node.js / TypeScript Errors

### Cannot find module

**Error Message:**
```
Error: Cannot find module '@/config' or its corresponding type declarations.
```

**Quick Fix:**
```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

---

### UnhandledPromiseRejection

**Error Message:**
```
(node:12345) UnhandledPromiseRejectionWarning: Error: Database connection failed
(node:12345) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated
```

**Quick Fix:**
```typescript
// ❌ WRONG
app.get('/users', async (req, res) => {
  const users = await db.getUsers();  // If this throws, crash!
  res.json(users);
});

// ✅ CORRECT
try {
  const users = await db.getUsers();
  res.json(users);
} catch (error) {
  res.status(500).json({ error: error.message });
}

// ✅ BETTER - Error handler middleware
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: 'Internal server error' });
});
```

---

### ECONNREFUSED

**Error Message:**
```
Error: connect ECONNREFUSED 127.0.0.1:5432
```

**Cause:** Cannot connect to database/service

**Quick Fix:**
```bash
# Check if service is running
lsof -i:5432  # PostgreSQL
# or
docker ps | grep postgres

# Start if needed
docker start postgres-container
# or
brew services start postgresql
```

---

## React Errors

### Cannot read property of undefined

**Error Message:**
```
TypeError: Cannot read property 'name' of undefined
```

**Quick Fix:**
```tsx
// ❌ WRONG
return <div>{user.name}</div>;

// ✅ CORRECT - Optional chaining
return <div>{user?.name}</div>;

// ✅ CORRECT - Default value
return <div>{user?.name ?? 'Unknown'}</div>;

// ✅ CORRECT - Conditional render
if (!user) return <div>Loading...</div>;
return <div>{user.name}</div>;
```

---

### Too many re-renders

**Error Message:**
```
Warning: Maximum update depth exceeded. This can happen when a component calls setState inside useEffect or another setState.
```

**Quick Fix:**
```tsx
// ❌ WRONG - Infinite loop
useEffect(() => {
  setCount(count + 1);  // Triggers re-render, triggers effect again!
}, []);

// ✅ CORRECT - Functional update
useEffect(() => {
  const timer = setInterval(() => {
    setCount(c => c + 1);  // Uses current value
  }, 1000);
  return () => clearInterval(timer);
}, []);

// ✅ CORRECT - Proper dependencies
useEffect(() => {
  setCount(initialValue);
}, [initialValue]);  // Only when initialValue changes
```

---

### useEffect runs infinitely

**Error Message:**
Component keeps re-rendering endlessly

**Cause:** Object/array in dependency array

**Quick Fix:**
```tsx
// ❌ WRONG - New object every render
useEffect(() => {
  fetchData(filters);
}, [filters]);  // filters object recreated every render!

// ✅ CORRECT - Primitive dependencies
useEffect(() => {
  fetchData({ category, status });
}, [category, status]);  // Primitives are stable

// ✅ CORRECT - Memoize object
const stableFilters = useMemo(() => filters, [filters.category, filters.status]);
useEffect(() => {
  fetchData(stableFilters);
}, [stableFilters]);
```

---

## Database Errors

### Connection Timeout

**Error Message:**
```
connection timeout
Pool was destroyed
```

**Quick Fix:**
```yaml
# Increase timeout
spring:
  datasource:
    hikari:
      connection-timeout: 30000  # 30 seconds
      maximum-pool-size: 20
```

---

### Deadlock

**Error Message:**
```
Deadlock found when trying to get lock; try restarting transaction
```

**Quick Fix:**
```java
@Transactional(timeout = 30)  // Add timeout
public void processOrder(Order order) {
    // Keep transactions short
    // Access resources in consistent order
}
```

---

## Build Errors

### Maven Compilation Error

**Error Message:**
```
[ERROR] COMPILATION ERROR
[ERROR] /path/to/File.java:[10,20] cannot find symbol
```

**Quick Fix:**
```bash
# Clean and rebuild
mvn clean compile

# Skip tests for faster build
mvn clean install -DskipTests

# Check for missing dependency
mvn dependency:tree | grep missing-library
```

---

### TypeScript Compilation Error

**Error Message:**
```
error TS2345: Argument of type 'string' is not assignable to parameter of type 'number'.
```

**Quick Fix:**
```typescript
// ❌ WRONG
function calculate(userId: string) { }
calculate(123);  // Error!

// ✅ CORRECT - Match types
function calculate(userId: string) { }
calculate("123");  // String

// ✅ CORRECT - Or use union type
function calculate(userId: string | number) { }
calculate(123);  // OK
calculate("123");  // OK
```

---

## Debugging Strategies

### 1. Read the Full Stack Trace

**Top of stack trace** = Where error originated
**Bottom of stack trace** = Where it was caught

```
Exception in thread "main" java.lang.NullPointerException  <-- Type
    at com.example.UserService.getName(UserService.java:25)  <-- Origin
    at com.example.UserController.getUser(UserController.java:15)
    at org.springframework.web...  <-- Framework calls
```

### 2. Add Logging

```java
// Add before error line
System.out.println("DEBUG: user = " + user);
log.info("Processing user: {}", userId);
```

### 3. Use Debugger

**IntelliJ IDEA:** Click left gutter to set breakpoint, then Debug (Shift+F9)
**VS Code:** Set breakpoint, then Run > Start Debugging (F5)

### 4. Isolate the Problem

```java
// Comment out code to find culprit
try {
    // step1();
    // step2();
    step3();  // If error here, problem isolated
} catch (Exception e) {
    e.printStackTrace();
}
```

---

## Emergency Fixes

### Application Won't Start

1. Check logs: `tail -f logs/application.log`
2. Check ports: `lsof -i:8080`
3. Check config: Verify application.yml/properties
4. Check database: `curl localhost:5432` or check container
5. Restart: Sometimes it's just stuck

### Tests Failing

1. Run single test: `mvn test -Dtest=UserServiceTest`
2. Clean build: `mvn clean test`
3. Check test data: Is database clean?
4. Check mocks: Are dependencies mocked correctly?

### Performance Issues

1. Check logs for slow queries
2. Enable profiling: `-Xprof` (Java) or clinic.js (Node)
3. Check memory: `htop` or Activity Monitor
4. Check database connections: Are they exhausted?

---

## Still Stuck?

1. **Search the error message** - Copy exact error into Google
2. **Check GitHub Issues** - Others likely had same problem
3. **Stack Overflow** - Search with technology tags
4. **Ask for help** - Provide:
   - Full error message
   - Relevant code snippet
   - What you've tried
   - Environment details (versions)
