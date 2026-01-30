---
name: e2e
description: Generate end-to-end tests for critical user flows using Playwright, Cypress, or REST Assured.
disable-model-invocation: true
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Edit
  - Write
---

# E2E Test Generation

Generate comprehensive end-to-end tests for critical user flows.

## Table of Contents

1. [Supported Frameworks](#supported-frameworks)
2. [Workflow](#workflow)
3. [Playwright with Page Object Model](#playwright-with-page-object-model)
4. [REST Assured (Java)](#rest-assured-java)
5. [Critical User Flows Checklist](#critical-user-flows-checklist)
6. [Flaky Test Management](#flaky-test-management)
7. [Best Practices](#best-practices)
8. [CI/CD Integration](#cicd-integration)
9. [Configuration](#configuration)
10. [Artifact Management](#artifact-management)

## Supported Frameworks

### React/Vue/Angular (Playwright - Recommended)
```bash
# Install
npm init playwright@latest

# Run tests
npx playwright test
npx playwright test --ui
npx playwright codegen localhost:3000

# Debug
npx playwright test --debug
npx playwright show-report
```

### Java API (REST Assured + Testcontainers)
```bash
mvn test -Dtest=*E2ETest
```

## Workflow

### 1. Identify Flow
What user journey needs testing? Examples:
- User registration → Email verification → Login
- Product search → Add to cart → Checkout → Payment
- Admin: Create item → Edit → Delete

### 2. Define Steps
List all user actions and expected outcomes:
```
1. Navigate to /register
2. Fill form (valid data)
3. Submit
4. Verify redirect to /verify-email
5. Check email sent (mock)
6. Click verification link
7. Verify redirect to /dashboard
```

### 3. Write Test

#### Playwright with Page Object Model
```typescript
// pages/RegisterPage.ts
export class RegisterPage {
  constructor(page) {
    this.page = page
    this.email = page.locator('[data-testid="email"]')
    this.password = page.locator('[data-testid="password"]')
    this.submit = page.locator('[data-testid="submit"]')
  }
  
  async goto() {
    await this.page.goto('/register')
  }
  
  async register(email, password) {
    await this.email.fill(email)
    await this.password.fill(password)
    await this.submit.click()
  }
}

// tests/auth/register.spec.ts
import { test, expect } from '@playwright/test'
import { RegisterPage } from '../pages/RegisterPage'

test.describe('User Registration', () => {
  test('should register with valid data', async ({ page }) => {
    const registerPage = new RegisterPage(page)
    
    // Arrange
    await registerPage.goto()
    
    // Act
    await registerPage.register('test@example.com', 'SecurePass123!')
    
    // Assert
    await expect(page).toHaveURL('/verify-email')
    await expect(page.locator('[data-testid="success"]')).toBeVisible()
    
    // Artifact
    await page.screenshot({ path: 'artifacts/register-success.png' })
  })
})
```

#### REST Assured (Java)
```java
@Testcontainers
class UserE2ETest {
    
    @Container
    static PostgreSQLContainer<?> postgres = 
        new PostgreSQLContainer<>("postgres:16");
    
    @Test
    void fullUserLifecycle() {
        // 1. Create user
        String userId = given()
            .contentType(ContentType.JSON)
            .body("{\"email\":\"test@example.com\",\"name\":\"Test\"}")
            .post("/api/users")
            .then()
            .statusCode(201)
            .extract().jsonPath().getString("id");
        
        // 2. Verify created
        given()
            .get("/api/users/" + userId)
            .then()
            .statusCode(200)
            .body("email", equalTo("test@example.com"));
        
        // 3. Update user
        given()
            .contentType(ContentType.JSON)
            .body("{\"name\":\"Updated\"}")
            .put("/api/users/" + userId)
            .then()
            .statusCode(200);
        
        // 4. Delete user
        given()
            .delete("/api/users/" + userId)
            .then()
            .statusCode(204);
        
        // 5. Verify deleted
        given()
            .get("/api/users/" + userId)
            .then()
            .statusCode(404);
    }
}
```

### 4. Run & Verify
```bash
# Run all tests
npx playwright test

# Run with trace for debugging
npx playwright test --trace on

# Run specific test
npx playwright test tests/auth/register.spec.ts
```

## Critical User Flows Checklist

### Authentication
- [ ] User registration (happy path)
- [ ] Registration validation errors
- [ ] Login with valid credentials
- [ ] Login with invalid credentials
- [ ] Password reset flow
- [ ] Logout

### Core Features
- [ ] Main CRUD operations (Create, Read, Update, Delete)
- [ ] Search functionality
- [ ] Filtering and sorting
- [ ] Pagination
- [ ] File uploads

### Payment (if applicable)
- [ ] Add payment method
- [ ] Process payment
- [ ] Handle payment failure
- [ ] Refund flow

### Error Handling
- [ ] 404 page
- [ ] 500 error handling
- [ ] Network failure recovery
- [ ] Form validation errors

### Edge Cases
- [ ] Empty states (no data)
- [ ] Long text inputs
- [ ] Special characters
- [ ] Mobile responsiveness

## Flaky Test Management

### Identify Flaky Tests
```bash
# Run test multiple times
npx playwright test --repeat-each=10

# Run with retries
npx playwright test --retries=3
```

### Common Causes & Fixes

**Race Conditions**
```typescript
// ❌ FLAKY
await page.click('[data-testid="button"]')

// ✅ STABLE (auto-wait)
await page.locator('[data-testid="button"]').click()
```

**Network Timing**
```typescript
// ❌ FLAKY
await page.waitForTimeout(5000)

// ✅ STABLE
await page.waitForResponse(resp => 
  resp.url().includes('/api/users') && resp.status() === 200
)
```

**Dynamic Content**
```typescript
// ❌ FLAKY
await expect(page.locator('[data-testid="result"]')).toBeVisible()

// ✅ STABLE
await page.locator('[data-testid="result"]').waitFor({ state: 'visible' })
await expect(page.locator('[data-testid="result"]')).toBeVisible()
```

### Quarantine Pattern
```typescript
// Mark as flaky
test('complex search query', async () => {
  test.fixme(true, 'Flaky - Issue #123')
  // test code
})

// Or skip in CI
test('complex search query', async () => {
  test.skip(process.env.CI, 'Flaky in CI - Issue #123')
  // test code
})
```

## Best Practices

### Selectors
- ✅ Use `data-testid` attributes (stable, semantic)
- ✅ Use semantic roles: `page.getByRole('button', { name: 'Submit' })`
- ❌ Avoid CSS classes (change with styling)
- ❌ Avoid XPath (brittle)

### Test Data
- Use unique data per test (timestamps, UUIDs)
- Reset database state before/after tests
- Use fixtures for reusable test data

### Test Independence
- Each test must pass independently
- No shared state between tests
- Clean up in `afterEach`

### Performance
- Keep tests under 30 seconds
- Use `parallel` execution where safe
- Mock slow external services

## CI/CD Integration

### GitHub Actions Example
```yaml
name: E2E Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: 18
          
      - name: Install dependencies
        run: npm ci
        
      - name: Install Playwright
        run: npx playwright install --with-deps
        
      - name: Run E2E tests
        run: npx playwright test
        
      - name: Upload artifacts
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: |
            playwright-report/
            test-results/
          retention-days: 30
```

## Playwright Configuration

```typescript
// playwright.config.ts
export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['junit', { outputFile: 'results.xml' }]
  ],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'mobile', use: { ...devices['Pixel 5'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
})
```

## Artifact Management

| Artifact | When to Capture | Configuration |
|----------|----------------|---------------|
| Screenshot | On failure | `screenshot: 'only-on-failure'` |
| Video | On failure | `video: 'retain-on-failure'` |
| Trace | Always | `trace: 'on-first-retry'` |

View trace: `npx playwright show-trace trace.zip`

## Output Format

```
## E2E Test Results

**Status:** ❌ FAILING (2 failed, 1 flaky)
**Duration:** 3m 45s

### Summary
| Suite | Total | Passed | Failed | Flaky |
|-------|-------|--------|--------|-------|
| Auth | 5 | 4 | 1 | 0 |
| Markets | 8 | 6 | 1 | 1 |
| Wallet | 3 | 3 | 0 | 0 |

### Failed Tests

#### 1. Auth - Password Reset
**File:** `tests/auth/reset.spec.ts:42`
**Error:** Element not found: [data-testid="reset-email"]
**Screenshot:** test-results/reset-failed.png
**Trace:** test-results/trace.zip

**Steps:**
1. Navigate to /reset-password
2. Fill email
3. Click submit
4. ❌ Element [data-testid="reset-email"] not found

**Fix:** Update selector to match actual element

#### 2. Markets - Search Results
**File:** `tests/markets/search.spec.ts:28`
**Error:** Expected 5 results, got 0
**Issue:** Search API returning empty array

### Flaky Tests

#### 1. Markets - Filter by Category
**File:** `tests/markets/filter.spec.ts:15`
**Flakiness:** 30% (3/10 runs failed)
**Action:** Quarantined - Issue #456

### Next Steps
- [ ] Fix Auth selector issue
- [ ] Investigate Markets search API
- [ ] Address flaky filter test
```

## Commands Reference

```bash
# Run tests
npx playwright test

# Run with UI
npx playwright test --ui

# Debug
npx playwright test --debug

# Generate test code
npx playwright codegen http://localhost:3000

# Show report
npx playwright show-report

# Update snapshots
npx playwright test --update-snapshots
```
