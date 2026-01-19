# Git Workflow

## Conventional Commits

### Formato
```
<type>[scope]: <description>

[body]

[footer]
```

### Types
| Type | Descripción |
|------|-------------|
| `feat` | Nueva funcionalidad |
| `fix` | Corrección de bug |
| `refactor` | Refactorización sin cambio de comportamiento |
| `docs` | Solo documentación |
| `test` | Añadir/modificar tests |
| `chore` | Tareas de mantenimiento |
| `style` | Formato, sin cambio de lógica |
| `perf` | Mejora de performance |

### Ejemplos
```bash
feat(auth): add JWT token refresh endpoint

fix(api): handle null response from external service

refactor(users): extract validation to separate class

docs: update API documentation for v2 endpoints

test(orders): add integration tests for payment flow

chore: update Spring Boot to 3.2.0
```

## Branching Strategy

### GitFlow Simplificado
```
main (producción)
  └── develop (desarrollo)
       ├── feature/user-auth
       ├── feature/payment-integration
       └── fix/login-timeout
```

### Nombres de Ramas
```
feature/<descripcion>   # Nueva funcionalidad
fix/<descripcion>       # Corrección de bug
hotfix/<descripcion>    # Fix urgente en producción
refactor/<descripcion>  # Refactorización
```

## Flujo de Trabajo

### Nueva Feature
```bash
# 1. Actualizar develop
git checkout develop
git pull origin develop

# 2. Crear rama
git checkout -b feature/user-profile

# 3. Desarrollar con commits pequeños
git add .
git commit -m "feat(users): add profile entity"

# 4. Push para PR
git push origin feature/user-profile

# 5. Crear Pull Request
```

### Pull Request Checklist
- [ ] Tests pasan
- [ ] Cobertura >= 80%
- [ ] Sin conflictos con develop
- [ ] Documentación actualizada
- [ ] Code review aprobado

## Comandos Útiles

```bash
# Estado
git status
git log --oneline -10

# Branches
git branch -a
git checkout -b new-branch
git branch -d branch-to-delete

# Stash
git stash
git stash pop
git stash list

# Rebase interactivo (limpiar commits)
git rebase -i HEAD~3

# Deshacer último commit (mantener cambios)
git reset --soft HEAD~1

# Deshacer cambios en archivo
git checkout -- file.txt

# Ver diferencias
git diff
git diff --staged
git diff branch1..branch2

# Blame
git blame file.txt
```

## Best Practices

1. **Commits pequeños y frecuentes**
2. **Mensajes descriptivos**
3. **Un commit = un cambio lógico**
4. **Pull antes de push**
5. **Nunca force push a main/develop**
6. **Review todo antes de merge**
7. **Mantener branches actualizadas**
