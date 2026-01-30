---
name: tdd
description: Test-Driven Development workflow. Write failing test first, then implement minimum code to pass.
disable-model-invocation: true
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Edit
  - Write
---

# TDD Workflow

Implement using strict Test-Driven Development.

## Process

### 1. RED - Write Failing Test First
- Write ONE test that describes expected behavior
- Run test - confirm it FAILS for the right reason
- Show test to user before proceeding

### 2. GREEN - Minimum Code to Pass
- Write the MINIMUM code to make test pass
- No extra features, no optimization
- Run test - confirm it PASSES

### 3. REFACTOR - Improve Code
- Clean up code while keeping tests green
- Apply patterns and best practices
- Run tests after each change

### 4. REPEAT
- Move to next requirement
- Write next failing test
- Continue cycle

## My Role

When asked to implement a feature:

1. **Ask for requirements** if not clear
2. **Write the test first** - show it to user
3. **Run the test** - confirm it fails
4. **Implement minimum code** to pass
5. **Refactor** if needed
6. **Repeat** for next requirement

## Rules

- NEVER write production code without a failing test
- ONE test at a time
- Tests must fail before implementation
- Keep cycles short (< 10 minutes each)

## Test Naming

```
methodName_scenario_expectedResult
```

Examples:
- `create_withValidData_returnsUser`
- `findById_notFound_throwsException`

## Test Quality Checks

- Test name follows: `method_scenario_expected`
- Test has Given-When-Then structure
- One concept per test
- No logic in tests
- Tests are independent

## Commands

```bash
# Java/Maven
mvn test -Dtest=ClassName#methodName

# Node.js/Vitest
npx vitest run --reporter=verbose

# Run specific test
npx vitest run path/to/test.ts
```

## Never Do

- Write production code before tests
- Write multiple tests before implementing
- Skip the refactor phase
- Leave tests incomplete

---

Start by asking: "What behavior should we test first?"
