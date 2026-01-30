---
name: refactor
description: Clean up and refactor code while maintaining functionality. Removes dead code, improves structure.
disable-model-invocation: true
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Edit
---

# Refactor & Clean

Clean up code while maintaining functionality.

## Scope

Specify what to refactor:
- A specific file
- A feature/module
- Dead code removal
- Pattern application

## Process

1. **Run tests first** - Ensure green baseline
2. **Identify issues** - List what needs improvement
3. **Refactor incrementally** - One change at a time
4. **Run tests after each change** - Never break functionality
5. **Final verification** - All tests pass

## Common Refactorings

### Dead Code Removal
- Unused imports
- Unused variables/methods
- Commented-out code
- Unreachable code

### Code Quality
- Extract long methods
- Remove duplication
- Improve naming
- Simplify conditionals

### Pattern Application
- Extract interface
- Apply dependency injection
- Use builder pattern
- Apply strategy pattern

## Rules

- ⚠️ NEVER refactor without tests
- ⚠️ One refactoring at a time
- ⚠️ Run tests after each change
- ⚠️ Commit frequently

## Output Format

### Baseline
- Tests: X passing, Y failing
- Files to refactor: list

### Changes Made
| File | Refactoring | Tests |
|------|-------------|-------|
| ... | ... | ✅ |

### Final State
- Tests: X passing, 0 failing
- Lines removed: N
- Complexity reduced: description
