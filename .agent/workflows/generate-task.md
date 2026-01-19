---
description: Generar tarea completa desde descripción breve
---

# /generate-task - Generar Tarea

## Uso
```
/generate-task <descripción breve de la tarea>
```

## Proceso

1. **Recibir** descripción del usuario
2. **Analizar** contexto del proyecto
3. **Expandir** en tarea completa
4. **Asignar** ID y prioridad
5. **Guardar** en `.agent/tasks/`
6. **Actualizar** QUEUE.md

## Agente
Delegar a: `.agent/agents/task-generator.md`

## Output

Archivo en `.agent/tasks/NNN_nombre.md` con:
- META (ID, prioridad, estado, deps, timeout)
- OBJETIVO
- CONTEXTO
- CRITERIOS DE ACEPTACIÓN
- PLAN DE EJECUCIÓN
- VALIDACIÓN

## Formato de Nombre
```
NNN_descripcion_breve.md

Ejemplo:
001_setup_inicial.md
002_implementar_auth.md
```

## Modo Batch
```
/generate-task
1. setup proyecto
2. implementar auth
3. crear API usuarios
4. añadir tests
```

Genera múltiples tareas con dependencias detectadas automáticamente.
