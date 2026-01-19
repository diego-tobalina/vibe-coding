---
description: Refactorizar y limpiar código
---

# /refactor - Refactorización

## Uso
```
/refactor <archivo o área a refactorizar>
```

## Proceso

1. **Verificar** que hay tests que pasen
2. **Identificar** code smells
3. **Planificar** refactorización
4. **Ejecutar** cambios pequeños
5. **Ejecutar tests** después de cada cambio
6. **Verificar** comportamiento idéntico

## Agente
Delegar a: `.agent/agents/refactor-cleaner.md`

## Code Smells a Buscar

- Métodos >50 líneas
- Clases >400 líneas
- Anidamiento >4 niveles
- Código duplicado
- Parámetros excesivos (>4)
- Números mágicos

## Refactorizaciones Seguras

- Extract Method
- Extract Class
- Rename
- Inline Variable
- Move Method

## Checklist
- [ ] Tests pasan antes
- [ ] Cambios pequeños
- [ ] Tests después de cada cambio
- [ ] Comportamiento idéntico
- [ ] Código más legible
