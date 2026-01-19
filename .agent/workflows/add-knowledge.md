---
description: Añadir, modificar o eliminar conocimiento del sistema de agentes
---

# /add-knowledge - Gestión de Conocimiento

## Uso
```
/add-knowledge <descripción del conocimiento a añadir/modificar/eliminar>
```

## Ejemplos
```
/add-knowledge añadir patrones de Redis para caching
/add-knowledge actualizar regla de cobertura a 90%
/add-knowledge eliminar skill de Angular (ya no lo usamos)
/add-knowledge añadir agente para desarrollo en Python
```

## Proceso

1. **Entender** qué conocimiento gestionar
2. **Identificar** archivos afectados
3. **Aplicar** cambios con formato coherente
4. **Verificar** que no rompe nada
5. **Reportar** cambios realizados

## Agente
Delegar a: `.agent/agents/knowledge-updater.md`

## Tipos de Conocimiento

| Tipo | Ubicación |
|------|-----------|
| Patrones/librerías | `.agent/skills/` |
| Reglas obligatorias | `.agent/rules/` |
| Comandos | `.agent/workflows/` |
| Agentes nuevos | `.agent/agents/` |
| Plantillas | `.agent/templates/` |

## Checklist
- [ ] Tipo de cambio claro (añadir/modificar/eliminar)
- [ ] Archivos identificados
- [ ] Formato consistente
- [ ] Referencias actualizadas
- [ ] Cambios reportados
