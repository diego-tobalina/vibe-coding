---
name: springboot-coding
description: Spring Boot coding guidelines for REST APIs, data access, caching, and configuration. This skill covers Spring-specific patterns. For core Java, use java-coding skill.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Edit
  - Write
---

# Spring Boot Coding Guidelines

**IMPORTANT:** This skill covers **Spring Boot framework patterns**. For core Java language, see `java-coding/`.

## Quick Start Checklist

Before writing Spring Boot code:
- [ ] Use constructor injection (never `@Autowired` on fields)
- [ ] Add `@Transactional(readOnly = true)` on service class
- [ ] Configure `open-in-view: false` in application.yml
- [ ] Use `@Valid` on request bodies
- [ ] Handle LazyInitializationException with JOIN FETCH
- [ ] Never expose stack traces in production

---

## Required Imports

```java
// Web & REST
import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;
import org.springframework.http.HttpStatus;
import org.springframework.validation.annotation.Validated;
import jakarta.validation.Valid;

// Data & JPA
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

// Transactions & Caching
import org.springframework.transaction.annotation.Transactional;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.SerializationPair;
import java.time.Duration;

// Security
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import static org.springframework.security.config.http.SessionCreationPolicy.STATELESS;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

// Async
import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.core.task.TaskExecutor;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.Executor;

// Exception Handling
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.method.annotation.MethodArgumentNotValidException;

// Utilities
import java.net.URI;
import java.time.Instant;
import java.util.Optional;
import java.util.UUID;
import java.util.stream.Collectors;

// Lombok
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
```

---

## Application Configuration

### application.yml (Production-Ready)

```yaml
spring:
  application:
    name: ${APP_NAME:my-service}
  
  # CRITICAL: Disable Open Session in View
  jpa:
    open-in-view: false
    hibernate:
      ddl-auto: validate  # Never use create/update in production
    properties:
      hibernate:
        jdbc.batch_size: 20
        jdbc.time_zone: UTC
        format_sql: true  # Only in dev
        highlight_sql: true  # Only in dev
        use_sql_comments: true  # Only in dev
  
  # Database connection pool
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 20000  # 20 seconds
      idle-timeout: 300000  # 5 minutes
      max-lifetime: 1200000  # 20 minutes
      leak-detection-threshold: 60000  # 1 minute
  
  # Migrations
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
  
  # Cache
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}
      timeout: 2000  # 2 seconds
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0

server:
  port: ${PORT:8080}
  error:
    include-stacktrace: never  # CRITICAL: Never expose in production
    include-message: always
    include-binding-errors: never

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
  health:
    db:
      enabled: true
    redis:
      enabled: true
```

---

## REST Controller

### Standard CRUD Controller

```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
@Validated
public class UserController {
    
    private final UserService userService;
    
    @GetMapping
    public ResponseEntity<Page<UserDto>> findAll(Pageable pageable) {
        return ResponseEntity.ok(userService.findAll(pageable));
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<UserDto> findById(@PathVariable UUID id) {
        return ResponseEntity.ok(userService.findById(id));
    }
    
    @PostMapping
    public ResponseEntity<UserDto> create(@Valid @RequestBody UserCreateDto dto) {
        UserDto created = userService.create(dto);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();
        return ResponseEntity.created(location).body(created);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<UserDto> update(
            @PathVariable UUID id,
            @Valid @RequestBody UserUpdateDto dto) {
        return ResponseEntity.ok(userService.update(id, dto));
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable UUID id) {
        userService.delete(id);
    }
}
```

---

## Service Layer

### Best Practice Pattern

```java
@Service
@RequiredArgsConstructor
@Slf4j
@Transactional(readOnly = true)  // Default to read-only
public class UserService {
    
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    
    public Page<UserDto> findAll(Pageable pageable) {
        return userRepository.findAll(pageable).map(userMapper::toDto);
    }
    
    @Cacheable(value = "users", key = "#id")
    public UserDto findById(UUID id) {
        return userRepository.findById(id)
            .map(userMapper::toDto)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
    
    @Transactional  // Override readOnly for write operations
    @CacheEvict(value = "users", key = "#result.id")
    public UserDto create(UserCreateDto dto) {
        if (userRepository.existsByEmail(dto.getEmail())) {
            throw new DuplicateEmailException(dto.getEmail());
        }
        User user = userMapper.toEntity(dto);
        return userMapper.toDto(userRepository.save(user));
    }
    
    @Transactional
    @CachePut(value = "users", key = "#id")
    public UserDto update(UUID id, UserUpdateDto dto) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
        userMapper.updateEntity(dto, user);
        return userMapper.toDto(userRepository.save(user));
    }
    
    @Transactional
    @CacheEvict(value = "users", key = "#id")
    public void delete(UUID id) {
        if (!userRepository.existsById(id)) {
            throw new UserNotFoundException(id);
        }
        userRepository.deleteById(id);
    }
}
```

---

## Repository Layer

### Basic Repository

```java
public interface UserRepository extends JpaRepository<User, UUID> {
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
    
    @Query("SELECT u FROM User u WHERE u.active = true")
    Page<User> findAllActive(Pageable pageable);
}
```

### Solving N+1 Problem with JOIN FETCH

```java
public interface UserRepository extends JpaRepository<User, UUID> {
    
    // ❌ WRONG - Causes N+1 queries
    @Query("SELECT u FROM User u")
    List<User> findAllWithOrders();  // Each user.orders triggers new query
    
    // ✅ CORRECT - JOIN FETCH loads everything in one query
    @Query("SELECT u FROM User u LEFT JOIN FETCH u.orders")
    List<User> findAllWithOrders();
    
    // ✅ CORRECT - For single entity with collections
    @Query("SELECT u FROM User u LEFT JOIN FETCH u.orders WHERE u.id = :id")
    Optional<User> findByIdWithOrders(@Param("id") UUID id);
    
    // ✅ CORRECT - EntityGraph approach (alternative)
    @EntityGraph(attributePaths = {"orders", "orders.items"})
    @Query("SELECT u FROM User u WHERE u.id = :id")
    Optional<User> findByIdWithOrdersGraph(@Param("id") UUID id);
}
```

---

## Common Spring Boot Mistakes

### Mistake 1: LazyInitializationException

**The Problem:**
```java
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    // ❌ WRONG - LazyInitializationException when accessing orders
    public List<Order> getUserOrders(UUID userId) {
        User user = userRepository.findById(userId).orElseThrow();
        return user.getOrders();  // Session closed! Exception thrown!
    }
}
```

**Why it happens:**
- JPA uses lazy loading by default for `@OneToMany` relationships
- The Hibernate Session closes after the transaction ends
- Accessing unloaded collections after session close throws exception

**Solutions:**

**Option 1: JOIN FETCH (Recommended)**
```java
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders WHERE u.id = :id")
Optional<User> findByIdWithOrders(@Param("id") UUID id);
```

**Option 2: Transactional boundary**
```java
@Service
@Transactional(readOnly = true)  // Extends transaction scope
public class UserService {
    
    public List<Order> getUserOrders(UUID userId) {
        User user = userRepository.findById(userId).orElseThrow();
        return user.getOrders();  // Works because session is still open
    }
}
```

**Option 3: Eager loading (NOT recommended)**
```java
@OneToMany(fetch = FetchType.EAGER)  // ❌ Bad for performance
private List<Order> orders;
```

### Mistake 2: N+1 Query Problem

**The Problem:**
```java
// ❌ WRONG - 101 queries for 100 users!
List<User> users = userRepository.findAll();  // 1 query
for (User user : users) {
    System.out.println(user.getOrders().size());  // 1 query per user = 100 queries
}
```

**Detection:**
Add to application.yml:
```yaml
spring:
  jpa:
    properties:
      hibernate:
        format_sql: true
logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.stat: DEBUG
```

**Solution:**
```java
// ✅ CORRECT - 1 query total
@Query("SELECT DISTINCT u FROM User u LEFT JOIN FETCH u.orders")
List<User> findAllWithOrders();

// Or use EntityGraph
@EntityGraph(attributePaths = "orders")
@Query("SELECT u FROM User u")
List<User> findAllWithOrders();
```

### Mistake 3: Open Session In View (OSIV)

**The Problem:**
OSIV keeps Hibernate session open during view rendering, hiding lazy loading issues but causing:
- Connection pool exhaustion
- Slow response times
- Memory leaks

**Configuration:**
```yaml
spring:
  jpa:
    open-in-view: false  # CRITICAL: Always disable
```

**Then handle lazy loading properly:**
```java
@Service
@Transactional(readOnly = true)
public class UserService {
    
    public UserDto getUser(UUID id) {
        // Load all needed data within transaction
        User user = userRepository.findByIdWithOrders(id)
            .orElseThrow(() -> new UserNotFoundException(id));
        return mapper.toDto(user);
    }
}
```

### Mistake 4: Field Injection

**The Problem:**
```java
@Service
public class UserService {
    @Autowired  // ❌ Field injection - hard to test, hidden dependencies
    private UserRepository userRepository;
    
    @Autowired
    private UserMapper userMapper;
}
```

**Why it's bad:**
- Cannot use final fields (immutable)
- Hidden dependencies - not clear what service needs
- Hard to unit test without Spring context
- Violates dependency injection best practices

**Solution:**
```java
@Service
@RequiredArgsConstructor  // Lombok generates constructor
public class UserService {
    private final UserRepository userRepository;  // final = immutable
    private final UserMapper userMapper;
}

// Equivalent to:
@Service
public class UserService {
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    
    public UserService(UserRepository userRepository, UserMapper userMapper) {
        this.userRepository = userRepository;
        this.userMapper = userMapper;
    }
}
```

### Mistake 5: Missing @Transactional

**The Problem:**
```java
@Service
public class OrderService {
    
    public Order createOrder(OrderDto dto) {
        Order order = new Order();
        // ... set fields
        orderRepository.save(order);  // Saved immediately
        
        orderItemRepository.saveAll(dto.getItems());  // Separate transaction!
        
        // If exception here, order saved but items not = inconsistent data
        updateInventory(dto.getItems());
    }
}
```

**Solution:**
```java
@Service
@RequiredArgsConstructor
public class OrderService {
    
    @Transactional  // All or nothing
    public Order createOrder(OrderDto dto) {
        Order order = orderRepository.save(new Order(dto));
        
        List<OrderItem> items = dto.getItems().stream()
            .map(item -> new OrderItem(order, item))
            .collect(Collectors.toList());
        orderItemRepository.saveAll(items);
        
        updateInventory(dto.getItems());
        
        return order;  // All committed together
    }
}
```

### Mistake 6: Wrong Transaction Propagation

**The Problem:**
```java
@Service
public class OrderService {
    
    @Transactional
    public void processOrder(Order order) {
        saveOrder(order);
        sendNotification(order);  // Waits for external service
        updateAnalytics(order);
    }
    
    @Transactional
    public void sendNotification(Order order) {
        // Calls external email service - takes 2-3 seconds
        emailService.send(order.getCustomerEmail());
    }
}
```

**Why it's bad:**
- Database connection held for 2-3 seconds while waiting for email
- Connection pool exhaustion under load
- Slow response times

**Solution:**
```java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final NotificationService notificationService;
    
    @Transactional
    public void processOrder(Order order) {
        saveOrder(order);
        updateAnalytics(order);
        // Transaction ends here - connection released
        
        // Async notification - no DB connection
        notificationService.sendOrderConfirmation(order);
    }
}

@Service
public class NotificationService {
    
    @Async  // Runs in separate thread
    public void sendOrderConfirmation(Order order) {
        emailService.send(order.getCustomerEmail());
    }
}
```

### Mistake 7: Exposing Internal Details in Errors

**The Problem:**
```yaml
server:
  error:
    include-stacktrace: always  # ❌ CRITICAL SECURITY ISSUE
```

**Why it's bad:**
- Stack traces reveal internal code structure
- Attackers can see library versions and vulnerabilities
- Database schema exposed in SQL exceptions

**Solution:**
```yaml
server:
  error:
    include-stacktrace: never  # Always in production
    include-message: always    # But show helpful messages
```

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        log.error("Unexpected error: {}", ex.getMessage(), ex);  // Log full error
        
        // But don't expose details to client
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred"));
    }
}
```

### Mistake 8: Not Using DTOs

**The Problem:**
```java
@RestController
public class UserController {
    
    @GetMapping("/{id}")
    public User findById(@PathVariable UUID id) {  // ❌ Exposing entity directly
        return userRepository.findById(id).orElseThrow();
    }
}
```

**Why it's bad:**
- Exposes internal entity structure
- Infinite recursion with bidirectional relationships
- Cannot control what data is returned
- Breaks if entity changes

**Solution:**
```java
@RestController
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;
    
    @GetMapping("/{id}")
    public UserDto findById(@PathVariable UUID id) {
        return userService.findById(id);  // Return DTO
    }
}

// DTO controls what is exposed
public record UserDto(
    UUID id,
    String email,
    String name,
    List<OrderSummaryDto> orders  // Not full Order entities
) {}
```

---

## Global Exception Handler

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusiness(BusinessException ex) {
        log.warn("Business error [{}]: {}", ex.getErrorCode(), ex.getMessage());
        return ResponseEntity
            .status(ex.getHttpStatus())
            .body(new ErrorResponse(ex.getErrorCode(), ex.getMessage()));
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return ResponseEntity
            .badRequest()
            .body(new ErrorResponse("VALIDATION_ERROR", message));
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        log.error("Unexpected error: {}", ex.getMessage(), ex);
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred"));
    }
}

public record ErrorResponse(String errorCode, String message, Instant timestamp) {
    public ErrorResponse(String errorCode, String message) {
        this(errorCode, message, Instant.now());
    }
}

// Business Exception
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
```

---

## Security Configuration

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())  // Disable if using JWT
            .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .anyRequest().authenticated())
            .httpBasic(Customizer.withDefaults());  // Or JWT
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

---

## Caching

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofHours(1))
            .serializeValuesWith(
                SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
        
        return RedisCacheManager.builder(factory)
            .cacheDefaults(config)
            .build();
    }
}

@Service
public class ProductService {
    
    @Cacheable(value = "products", key = "#id")
    public Product getProduct(UUID id) {
        return productRepository.findById(id).orElseThrow();
    }
    
    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(UUID id) {
        productRepository.deleteById(id);
    }
    
    @CacheEvict(value = "products", allEntries = true)
    public void clearCache() {
        // Clear all product cache
    }
}
```

---

## Async Operations

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class NotificationService {
    
    @Async
    public CompletableFuture<Void> sendEmailAsync(String to, String subject, String body) {
        emailClient.send(to, subject, body);
        return CompletableFuture.completedFuture(null);
    }
}
```

---

## Common AI Coding Mistakes (Autonomous Mode)

**When coding without user feedback, avoid these AI-specific errors:**

### 1. LazyInitializationException

**The Mistake:** Accessing lazy-loaded collections outside transaction
```java
@Service
public class UserService {
    @Autowired
    private UserRepository repo;
    
    // ❌ WRONG - Session closed when accessing orders
    public List<Order> getOrders(UUID id) {
        User user = repo.findById(id).orElseThrow();
        return user.getOrders();  // LazyInitializationException!
    }
}

// ✅ CORRECT - Use JOIN FETCH
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders WHERE u.id = :id")
Optional<User> findByIdWithOrders(@Param("id") UUID id);

// ✅ CORRECT - Transactional method
@Transactional(readOnly = true)
public List<Order> getOrders(UUID id) {
    User user = repo.findById(id).orElseThrow();
    return user.getOrders();  // Works - session still open
}
```

### 2. Field Injection (@Autowired on fields)

**The Mistake:** Using field injection instead of constructor
```java
@Service
public class UserService {
    @Autowired  // ❌ Field injection - hard to test
    private UserRepository repo;
}

// ✅ CORRECT - Constructor injection
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository repo;  // final = immutable
}
```

### 3. Missing @Transactional

**The Mistake:** Service method without transaction
```java
@Service
public class OrderService {
    // ❌ WRONG - No transaction
    public void createOrder(OrderDto dto) {
        Order order = orderRepo.save(new Order(dto));
        // If exception here, order saved but items not = inconsistent
        orderItemRepo.saveAll(dto.getItems());
    }
}

// ✅ CORRECT - @Transactional
@Transactional
public void createOrder(OrderDto dto) {
    Order order = orderRepo.save(new Order(dto));
    orderItemRepo.saveAll(dto.getItems());
    // Both succeed or both fail
}
```

### 4. N+1 Query Problem

**The Mistake:** Not using JOIN FETCH for collections
```java
// ❌ WRONG - N+1 queries
List<User> users = repo.findAll();  // 1 query
for (User u : users) {
    System.out.println(u.getOrders().size());  // N queries!
}

// ✅ CORRECT - Single query
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders")
List<User> findAllWithOrders();
```

### 5. Autonomous Decision Checklist

**Before generating code, verify:**

- [ ] Constructor injection (not field injection)
- [ ] @Transactional on write operations
- [ ] JOIN FETCH for entity collections
- [ ] open-in-view disabled in config
- [ ] DTOs returned from controllers (not entities)
- [ ] UUIDs used for IDs (not sequential)

**When uncertain about a Spring feature:**
```java
// Add a comment documenting uncertainty
// UNCERTAIN: Not sure if @Transactional is needed here
// ASSUMPTION: Adding it because method does multiple DB writes
// REVIEW: Remove if this is just a single read operation
@Transactional
public void processOrder(Order order) {
    // ...
}
```

---

## Production Checklist

Before deploying to production:

- [ ] `spring.jpa.open-in-view=false`
- [ ] `server.error.include-stacktrace=never`
- [ ] Database connection pool configured (HikariCP)
- [ ] Flyway or Liquibase for migrations (not ddl-auto)
- [ ] Health checks enabled (`/actuator/health`)
- [ ] Logging configured (JSON format)
- [ ] Caching configured for frequently accessed data
- [ ] Rate limiting implemented
- [ ] Input validation on all endpoints
- [ ] Proper error handling (no stack traces to client)
- [ ] Security headers configured
- [ ] HTTPS only
- [ ] Secrets in environment variables (never in code)
