---
name: logging
description: Logging best practices for Java (SLF4J/Logback) and Node.js (Winston/Pino). Use for structured logging, log levels, and avoiding sensitive data exposure.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Logging Guidelines

Structured logging best practices for Java and Node.js applications.

## ⚠️ CRITICAL: What NEVER to Log

**Logging sensitive data is a security breach. Never log:**

### Authentication & Credentials
```java
// ❌ NEVER DO THIS - Passwords
log.info("User login: email={}, password={}", email, password);

// ❌ NEVER DO THIS - Tokens
log.debug("JWT Token: {}", jwtToken);
log.info("Authorization: {}", request.getHeader("Authorization"));

// ❌ NEVER DO THIS - API Keys
log.error("API call failed with key: {}", apiKey);

// ✅ CORRECT - Log identifiers only
log.info("User login attempt: email={}", maskEmail(email));
log.debug("Token issued for userId={}", userId);
```

### Personal Identifiable Information (PII)
```java
// ❌ NEVER DO THIS - Full PII
log.info("User data: {}", user);  // User object contains SSN, DOB, address

// ❌ NEVER DO THIS - Financial data
log.info("Payment processed: card={}, cvv={}", cardNumber, cvv);

// ❌ NEVER DO THIS - Personal details
log.info("Customer: ssn={}, dob={}, phone={}", ssn, dateOfBirth, phone);

// ✅ CORRECT - Log only what you need
log.info("Payment processed: userId={}, amount={}, last4={}", 
         userId, amount, maskCardLast4(cardNumber));
```

### Health & Sensitive Personal Data
- Medical records or health information
- Biometric data (fingerprints, facial recognition)
- Genetic information
- Criminal history
- Sexual orientation or religious beliefs

### Masking Helper Methods
```java
public class LogMasker {
    public static String maskEmail(String email) {
        if (email == null || !email.contains("@")) return email;
        String[] parts = email.split("@");
        String name = parts[0];
        String domain = parts[1];
        String maskedName = name.length() > 2 
            ? name.substring(0, 2) + "***" 
            : "***";
        return maskedName + "@" + domain;
    }
    
    public static String maskCard(String cardNumber) {
        if (cardNumber == null || cardNumber.length() < 4) return "****";
        return "****-****-****-" + cardNumber.substring(cardNumber.length() - 4);
    }
    
    public static String maskSsn(String ssn) {
        if (ssn == null || ssn.length() < 4) return "***-**-****";
        return "***-**-" + ssn.substring(ssn.length() - 4);
    }
}

// Usage
log.info("User registered: email={}, ssn={}", 
         LogMasker.maskEmail(email), 
         LogMasker.maskSsn(ssn));
// Output: User registered: email=jo***@example.com, ssn=***-**-1234
```

### Node.js Redaction with Pino
```typescript
import pino from 'pino';

const logger = pino({
  redact: {
    paths: [
      'password', 
      '*.password',  // password field at any depth
      'token',
      'authorization',
      'apiKey',
      'ssn',
      'creditCard',
      'cvv'
    ],
    remove: true,  // Completely remove (not replace with [Redacted])
  },
});

// This will NOT log the password
logger.info({ username: 'john', password: 'secret123' });
// Output: {"username":"john"}
```

## Core Principles

1. **Use appropriate log levels** - Don't spam ERROR for info
2. **Never log sensitive data** - No passwords, tokens, PII
3. **Include context** - correlation IDs, user IDs, request paths
4. **Structured format** - JSON for production, human-readable for dev
5. **Log at boundaries** - Entry/exit of services, external calls

---

## Java Logging (SLF4J + Logback)

### Dependencies

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
</dependency>
<!-- For structured logging -->
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

### Basic Usage

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class UserService {
    private static final Logger log = LoggerFactory.getLogger(UserService.class);
    
    public UserDto createUser(UserCreateDto dto) {
        log.info("Creating user with email: {}", dto.getEmail());
        
        try {
            User user = userRepository.save(mapper.toEntity(dto));
            log.info("User created successfully: id={}", user.getId());
            return mapper.toDto(user);
        } catch (DataIntegrityException e) {
            log.warn("Failed to create user - duplicate email: {}", dto.getEmail());
            throw new DuplicateEmailException(dto.getEmail());
        } catch (Exception e) {
            log.error("Unexpected error creating user: {}", dto.getEmail(), e);
            throw e;
        }
    }
}
```

### Log Levels

| Level | Use For | Example |
|-------|---------|---------|
| **ERROR** | Failures requiring immediate attention | Database connection lost, payment failed |
| **WARN** | Unexpected but handled situations | Retry attempt, deprecated API usage |
| **INFO** | Significant business events | User registered, order placed |
| **DEBUG** | Detailed flow for troubleshooting | Method entry/exit, variable values |
| **TRACE** | Very detailed diagnostic | SQL queries, HTTP headers |

### Structured Logging (JSON)

```xml
<!-- logback-spring.xml -->
<configuration>
    <!-- Development: Human readable -->
    <springProfile name="dev">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>
    </springProfile>
    
    <!-- Production: JSON -->
    <springProfile name="prod">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder class="net.logstash.logback.encoder.LogstashEncoder">
                <includeMdcKeyName>correlationId</includeMdcKeyName>
                <includeMdcKeyName>userId</includeMdcKeyName>
            </encoder>
        </appender>
    </springProfile>
    
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

```java
// Add context with MDC (Mapped Diagnostic Context)
import org.slf4j.MDC;

@Component
public class CorrelationIdFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                     HttpServletResponse response, 
                                     FilterChain chain) throws ServletException, IOException {
        String correlationId = request.getHeader("X-Correlation-Id");
        if (correlationId == null) {
            correlationId = UUID.randomUUID().toString();
        }
        
        MDC.put("correlationId", correlationId);
        try {
            chain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

### What NOT to Log

```java
// ❌ WRONG - Never log sensitive data
log.info("User login: email={}, password={}", email, password);
log.debug("Credit card: {}", creditCardNumber);
log.info("Token: {}", jwtToken);

// ❌ WRONG - Don't log at ERROR for expected exceptions
try {
    userService.findById(id);
} catch (UserNotFoundException e) {
    log.error("User not found", e); // Should be WARN
    throw e;
}

// ✅ CORRECT
log.info("User login attempt: email={}", maskEmail(email));
log.debug("Processing payment for user={}", userId);

// Mask sensitive data
private String maskEmail(String email) {
    if (email == null || !email.contains("@")) return email;
    String[] parts = email.split("@");
    return parts[0].substring(0, Math.min(2, parts[0].length())) + "***@" + parts[1];
}
```

---

## Node.js Logging (Winston/Pino)

### Winston Setup

```bash
npm install winston
```

```typescript
// src/config/logger.ts
import winston from 'winston';

const { combine, timestamp, json, errors, printf } = winston.format;

// Development format
const devFormat = printf(({ level, message, timestamp, ...metadata }) => {
    return `${timestamp} [${level}]: ${message} ${
        Object.keys(metadata).length ? JSON.stringify(metadata) : ''
    }`;
});

export const logger = winston.createLogger({
    level: process.env.LOG_LEVEL || 'info',
    defaultMeta: { 
        service: process.env.APP_NAME || 'app',
        environment: process.env.NODE_ENV 
    },
    format: combine(
        timestamp(),
        errors({ stack: true }),
        process.env.NODE_ENV === 'production' ? json() : devFormat
    ),
    transports: [
        new winston.transports.Console()
    ],
});

// Add correlation ID
export const childLogger = (correlationId: string, userId?: string) => {
    return logger.child({ correlationId, userId });
};
```

### Usage in Express

```typescript
// src/middleware/requestLogger.ts
import { Request, Response, NextFunction } from 'express';
import { v4 as uuidv4 } from 'uuid';
import { childLogger } from '../config/logger';

export function requestLogger(req: Request, res: Response, next: NextFunction) {
    const correlationId = req.headers['x-correlation-id'] as string || uuidv4();
    req.correlationId = correlationId;
    req.log = childLogger(correlationId, req.user?.id);
    
    req.log.info({
        method: req.method,
        path: req.path,
        query: req.query,
    }, 'Request started');
    
    const start = Date.now();
    res.on('finish', () => {
        const duration = Date.now() - start;
        req.log.info({
            method: req.method,
            path: req.path,
            statusCode: res.statusCode,
            duration,
        }, 'Request completed');
    });
    
    next();
}

// Extend Express Request
declare global {
    namespace Express {
        interface Request {
            correlationId: string;
            log: ReturnType<typeof childLogger>;
        }
    }
}
```

```typescript
// src/services/user.service.ts
export async function createUser(data: CreateUserDto, req: Request) {
    req.log.info({ email: data.email }, 'Creating user');
    
    try {
        const user = await userRepository.create(data);
        req.log.info({ userId: user.id }, 'User created');
        return user;
    } catch (error) {
        req.log.error({ error, email: data.email }, 'Failed to create user');
        throw error;
    }
}
```

### Pino (High Performance Alternative)

```bash
npm install pino pino-http
```

```typescript
// src/config/logger.ts
import pino from 'pino';

export const logger = pino({
    level: process.env.LOG_LEVEL || 'info',
    transport: process.env.NODE_ENV !== 'production' 
        ? { target: 'pino-pretty' }
        : undefined,
    base: {
        service: process.env.APP_NAME,
    },
});

// Express middleware
import pinoHttp from 'pino-http';
app.use(pinoHttp({ logger }));
```

### What NOT to Log (Node.js)

```typescript
// ❌ WRONG
logger.info('User login', { email, password }); // Never log passwords!
logger.debug('Processing payment', { cardNumber, cvv });
logger.info({ authorization: req.headers.authorization }); // Never log tokens!

// ✅ CORRECT
logger.info('User login attempt', { email: maskEmail(email) });
logger.debug('Processing payment', { userId: req.user.id });
```

---

## Best Practices Summary

### Always Do
- ✅ Log at service boundaries (entry/exit)
- ✅ Include correlation IDs for tracing
- ✅ Log context (userId, requestId) not just messages
- ✅ Use appropriate log levels
- ✅ Structure logs as JSON in production
- ✅ Include timing information for external calls

### Never Do
- ❌ Log passwords, tokens, credit cards, SSNs
- ❌ Log PII (email, phone, address) at INFO/ERROR
- ❌ Use ERROR level for expected exceptions
- ❌ Log at DEBUG in production (performance)
- ❌ Log large objects/arrays completely
- ❌ Use string concatenation in log messages

### Log Entry/Exit Pattern

```java
// Java
public Order processOrder(OrderRequest request) {
    log.info("Processing order: userId={}, items={}", 
             request.getUserId(), 
             request.getItems().size());
    
    long start = System.currentTimeMillis();
    try {
        Order order = orderService.create(request);
        log.info("Order processed: orderId={}, durationMs={}", 
                 order.getId(), 
                 System.currentTimeMillis() - start);
        return order;
    } catch (Exception e) {
        log.error("Order processing failed: userId={}, error={}", 
                 request.getUserId(), 
                 e.getMessage(), 
                 e);
        throw e;
    }
}
```

```typescript
// Node.js
async function processOrder(request: OrderRequest, req: Request): Promise<Order> {
    req.log.info({ userId: request.userId, itemCount: request.items.length }, 'Processing order');
    
    const start = Date.now();
    try {
        const order = await orderService.create(request);
        req.log.info({ orderId: order.id, duration: Date.now() - start }, 'Order processed');
        return order;
    } catch (error) {
        req.log.error({ error, userId: request.userId }, 'Order processing failed');
        throw error;
    }
}
```
