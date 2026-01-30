---
description: Git conventions for commits, branches, and pull requests
---

# Git Workflow Rules

## Commit Messages

Format: `type(scope): description`

Types:
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code restructuring
- `test`: Adding/modifying tests
- `docs`: Documentation changes
- `chore`: Maintenance tasks

Examples:
- `feat(user): add email validation`
- `fix(order): correct price calculation`

## Branch Strategy

- `main`: Production-ready code
- `develop`: Integration branch
- `feature/*`: New features
- `fix/*`: Bug fixes

## Pull Request Guidelines

- Keep PRs small and focused
- Include tests for changes
- Update documentation if needed
- Request review before merge
