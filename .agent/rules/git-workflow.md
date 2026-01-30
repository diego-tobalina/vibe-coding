---
name: git-workflow
description: Git conventions for commits, branches, and pull requests. Apply when making commits or working with git.
mode: model_decision
---

# Git Workflow Rules

## Commit Messages

Format: `type(scope): description`

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code restructuring
- `test`: Adding/modifying tests
- `docs`: Documentation changes
- `chore`: Maintenance tasks

**Examples:**
- `feat(user): add email validation`
- `fix(order): correct price calculation`
- `refactor(auth): extract token service`

## Branch Strategy

- `main`: Production-ready code
- `develop`: Integration branch
- `feature/*`: New features (e.g., `feature/user-auth`)
- `fix/*`: Bug fixes (e.g., `fix/login-error`)

## Pull Request Guidelines

- Keep PRs small and focused (< 400 lines)
- Include tests for changes
- Update documentation if needed
- Request review before merge
- Write descriptive PR title following commit format

## Pre-Commit Checklist

Before committing:
- [ ] Code follows naming conventions
- [ ] Tests pass locally
- [ ] No commented-out code left
- [ ] Commit message follows format
