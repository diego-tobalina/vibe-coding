---
description: Revisar código por calidad, seguridad y mantenibilidad
---

# /code-review - Revisión de Código

## Uso
```
/code-review [archivo o directorio opcional]
```

## Proceso

1. **Ejecutar** `git diff` para ver cambios
2. **Revisar** seguridad (secretos, injection, etc.)
3. **Verificar** calidad (tamaño, naming, errores)
4. **Evaluar** performance
5. **Reportar** issues por prioridad

## Agente
Delegar a: `.agent/agents/code-reviewer.md`

## Categorías

| Prioridad | Acción |
|-----------|--------|
| CRÍTICO | Bloquear, arreglar ya |
| ALTO | Arreglar antes de merge |
| MEDIO | Arreglar cuando sea posible |
| BAJO | Sugerencia |

## Checklist
- [ ] Sin secretos hardcodeados
- [ ] Inputs validados
- [ ] Errores manejados
- [ ] Tests incluidos
- [ ] Naming claro
- [ ] Sin código muerto
