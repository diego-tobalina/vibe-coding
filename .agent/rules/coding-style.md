---
name: coding-style
description: Universal coding style rules for naming conventions, code structure, and error handling. Apply to all Java, TypeScript, JavaScript, and Python files.
mode: always_on
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

## Autonomous Decision Making

When coding without user feedback, use this confidence framework:

| Level | Confidence | Action | Examples |
|-------|-----------|--------|----------|
| **1** | 95-100% | ✅ Proceed confidently | Adding standard CRUD endpoint, simple validation |
| **2** | 80-94% | ✅ Proceed with safe default | Tech choice, naming conventions |
| **3** | 60-79% | ⚠️ Proceed with comment | Document assumption in comment |
| **4** | 40-59% | ⚠️ Conservative + detailed comment | Add UNCERTAIN comment |
| **5** | <40% | 🛑 STOP - Cannot proceed | Requirements contradictory |

## Naming Conventions

| Type | Convention | Example | ❌ WRONG | ✅ CORRECT |
|------|------------|---------|----------|------------|
| Variable | camelCase | userEmail | user_email | userEmail |
| Constant | UPPER_SNAKE | MAX_COUNT | maxCount | MAX_COUNT |
| Function | camelCase | calculateTotal() | calc_total() | calculateTotal() |
| Class | PascalCase | UserService | userService | UserService |
| Boolean | is/has/can | isValid | valid | isValid |
| Collection | Plural | users | userList | users |

## Code Structure

**Maximum Sizes:**
- Function/Method: 20 lines
- Class: 400 lines
- File: 500 lines
- Nesting: 4 levels max
- Parameters: 4 max (use builder/DTO)

## Error Handling

**Fail Fast:**
```java
// ❌ WRONG - Silent failure
if (order == null) { return; }

// ✅ CORRECT
if (order == null) {
    throw new IllegalArgumentException("Order cannot be null");
}
```

## Code Formatting

- Indentation: 4 spaces (Java), 2 spaces (JS/TS)
- Always use braces, even for single line
- Maximum line length: 120 characters
- Space after keywords: `if (condition)` not `if(condition)`

## Comments

**Comment WHY, not WHAT:**
```java
// ❌ WRONG
// Increment counter
counter++;

// ✅ CORRECT
// Retry 3 times because payment gateway occasionally times out
int maxRetries = 3;
```
