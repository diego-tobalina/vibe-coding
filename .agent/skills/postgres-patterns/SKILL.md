---
name: postgres-patterns
description: PostgreSQL patterns with JPA, Flyway migrations, and query optimization. Use when working with PostgreSQL databases.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# PostgreSQL Patterns

## Entity Design

```java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_user_email", columnList = "email"),
    @Index(name = "idx_user_created_at", columnList = "created_at")
})
@Getter
@Setter
@NoArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Column(nullable = false)
    private String name;
    
    @Column(name = "created_at", nullable = false, updatable = false)
    private Instant createdAt = Instant.now();
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
    
    @Version
    private Long version;
}
```

## Repository Patterns

```java
public interface UserRepository extends JpaRepository<User, UUID> {
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
    
    @Query("SELECT u FROM User u WHERE u.active = true ORDER BY u.createdAt DESC")
    Page<User> findAllActive(Pageable pageable);
    
    @Query(value = "SELECT * FROM users WHERE created_at > :since", nativeQuery = true)
    List<User> findCreatedAfter(@Param("since") Instant since);
    
    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.lastLogin < :threshold")
    int deactivateInactiveUsers(@Param("threshold") Instant threshold);
}
```

## Flyway Migrations

```sql
-- V1__create_users_table.sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL,
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE,
    version BIGINT DEFAULT 0
);

CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_user_created_at ON users(created_at DESC);
CREATE INDEX idx_user_active ON users(active) WHERE active = true;
```

## Connection Pool (HikariCP)

```yaml
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:5432/${DB_NAME}
    username: ${DB_USER}
    password: ${DB_PASSWORD}
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 20000
      idle-timeout: 300000
      max-lifetime: 1200000
```

## Query Optimization

```java
// Avoid N+1: Use JOIN FETCH
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders WHERE u.id = :id")
Optional<User> findByIdWithOrders(@Param("id") UUID id);

// Pagination with count query optimization
@Query(value = "SELECT u FROM User u WHERE u.active = true",
       countQuery = "SELECT COUNT(u.id) FROM User u WHERE u.active = true")
Page<User> findAllActivePaged(Pageable pageable);

// Projection for read-only queries
public interface UserSummary {
    UUID getId();
    String getName();
    String getEmail();
}

@Query("SELECT u.id as id, u.name as name, u.email as email FROM User u")
List<UserSummary> findAllSummaries();
```

## Batch Operations

```java
// Batch insert with JPA
@Transactional
public void saveAll(List<User> users) {
    int batchSize = 50;
    for (int i = 0; i < users.size(); i++) {
        entityManager.persist(users.get(i));
        if (i % batchSize == 0) {
            entityManager.flush();
            entityManager.clear();
        }
    }
}

// Bulk update (more efficient than loading entities)
@Modifying
@Query("UPDATE User u SET u.active = false WHERE u.lastLogin < :threshold")
int deactivateInactive(@Param("threshold") Instant threshold);
```

## Common Migrations

```sql
-- V2__add_column.sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- V3__add_index.sql
CREATE INDEX CONCURRENTLY idx_user_phone ON users(phone);

-- V4__add_table.sql
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    total NUMERIC(10,2) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

## Best Practices

- Use UUIDs for primary keys (`gen_random_uuid()`)
- Add indexes for frequently queried columns
- Use `@Version` for optimistic locking
- Prefer JPQL, use native queries for complex operations
- Always include `WHERE` clause indexes
- Use partial indexes for filtered queries
- Use `CONCURRENTLY` for production index creation
- Set appropriate pool sizes based on load
