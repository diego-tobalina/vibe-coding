---
description: Crear plan de implementación para features complejas
---

# /plan - Planificación de Implementación

## Uso
```
/plan <descripción del feature o cambio>
```

## Proceso

1. **Analizar** requisitos y contexto
2. **Identificar** componentes afectados
3. **Diseñar** solución técnica
4. **Desglosar** en pasos atómicos
5. **Documentar** dependencias y riesgos
6. **Crear** tareas en `.agent/tasks/`

## Agente
Delegar a: `.agent/agents/planner.md`

## Output
Genera un plan en formato:
- Resumen
- Requisitos
- Cambios por componente
- Pasos de implementación
- Estrategia de testing
- Riesgos y mitigaciones

## Checklist
- [ ] Requisitos claros
- [ ] Componentes identificados
- [ ] Pasos atómicos definidos
- [ ] Dependencias mapeadas
- [ ] Tests planificados
