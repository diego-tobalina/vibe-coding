# Gestión de Memoria en Conversaciones Largas

## El Problema

Los modelos de IA tienen **contexto limitado**. En conversaciones largas:
- Se pierde información del principio
- Se repiten errores ya resueltos
- Se olvidan decisiones tomadas
- Se descontextualiza el trabajo

## Solución: Sistema de Archivos de Memoria

```
.agent/
├── CONTEXT.md      # Estado actual del proyecto
├── LEARNINGS.md    # Errores resueltos y patrones
├── QUEUE.md        # Tareas pendientes
└── LOGS.md         # Historial de acciones
```

## Cuándo Actualizar

| Evento | Archivo a Actualizar |
|--------|----------------------|
| Descubrir estructura del proyecto | CONTEXT.md |
| Resolver un error nuevo | LEARNINGS.md |
| Cambiar de tarea | QUEUE.md |
| Completar acción significativa | LOGS.md |
| Decisión de arquitectura | CONTEXT.md |

## CONTEXT.md - Mapa del Proyecto

### Actualizar cuando:
- Se descubre nueva información del proyecto
- Cambia la arquitectura
- Se añaden/quitan dependencias importantes

### Formato:
```markdown
# PROJECT CONTEXT

## Entorno
- **OS:** macOS
- **Java:** 17
- **Build:** Maven

## Arquitectura
- Backend: Spring Boot 3.x
- DB: PostgreSQL 15
- Cache: Redis

## Estructura
src/main/java/com/example/
├── controller/
├── service/
├── repository/
└── entity/

## Decisiones Importantes
- Usar MapStruct para mapeos
- DTOs inmutables con Lombok @Value
```

## LEARNINGS.md - Base de Conocimiento

### Actualizar cuando:
- Se resuelve un error nuevo
- Se descubre un workaround
- Se encuentra un patrón útil

### Formato:
```markdown
# KNOWLEDGE BASE

## [2024-01-15] Error: Bean Not Found
**Síntoma:** `No qualifying bean of type 'X' available`
**Causa:** Faltaba @Service en la clase
**Fix:** Añadir @Service o @Component

## [2024-01-16] Workaround: Lombok + MapStruct
**Problema:** Conflicto entre annotation processors
**Solución:** Añadir lombok-mapstruct-binding
```

## QUEUE.md - Estado de Tareas

### Actualizar cuando:
- Se inicia una tarea
- Se completa una tarea
- Se bloquea una tarea

### Formato:
```markdown
# EXECUTION QUEUE

## Métricas
- Total: 5 | Completadas: 2 | Pendientes: 3

## Tarea Activa
- **Archivo:** 003_implement_auth.md
- **Inicio:** 2024-01-15 10:30

## Pendientes
- [ ] 004_add_tests.md
- [ ] 005_deploy.md
```

## LOGS.md - Historial

### Actualizar cuando:
- Se completa cualquier paso significativo
- Hay errores recuperados
- Hay decisiones tomadas

### Formato:
```markdown
# EXECUTION LOG

## 2024-01-15

### 10:30 | TASK_START | 003_implement_auth
### 10:45 | CODE_CREATED | AuthController.java
### 11:00 | ERROR | Bean conflict
### 11:05 | FIXED | Added @Lazy annotation
### 11:30 | TASK_COMPLETE | Auth implemented
```

## Estrategias de Compresión

### 1. Resumir en lugar de copiar
```markdown
# ❌ Mal
Ejecuté mvn clean install y el output fue...
[500 líneas de output]

# ✅ Bien
Build exitoso en 45s. 150 tests passed.
```

### 2. Mantener solo lo relevante
```markdown
# ❌ Mal
- Abrí el archivo
- Leí la línea 1
- Leí la línea 2...

# ✅ Bien
- UserService.java: Implementado createUser()
```

### 3. Archivar información antigua
```markdown
# En LEARNINGS.md
## Archivo (problemas ya no relevantes)
[Mover errores de versiones antiguas]
```

## Patrón de Sincronización

Al inicio de cada sesión:
```
1. LOAD .agent/CONTEXT.md
   → Entender el proyecto
   
2. LOAD .agent/LEARNINGS.md
   → Recordar errores pasados
   
3. LOAD .agent/QUEUE.md
   → Saber qué tarea sigue
   
4. RESUME desde último estado
```

## Ejemplo de Flujo

```
Usuario: "Continúa con la implementación"

Agente:
1. Lee QUEUE.md → Tarea activa: 003_implement_auth
2. Lee tarea 003 → Ve que está en paso 3/5
3. Lee LEARNINGS.md → Recuerda que MapStruct necesita config especial
4. Continúa desde paso 3
5. Al terminar paso → Actualiza QUEUE.md
6. Si hay error nuevo → Documenta en LEARNINGS.md
```

## Anti-patrones

### ❌ No hacer
1. Guardar código completo en memoria
2. Logs extremadamente detallados
3. No actualizar nunca los archivos
4. Duplicar información entre archivos

### ✅ Hacer
1. Referencias a archivos, no copias
2. Logs concisos con timestamp
3. Actualizar al completar cada paso
4. Un propósito claro por archivo

## Comandos de Memoria

```bash
# Ver estado actual
cat .agent/QUEUE.md

# Ver últimos errores
tail -20 .agent/LEARNINGS.md

# Ver actividad reciente
tail -50 .agent/LOGS.md

# Buscar error específico
grep -i "nullpointer" .agent/LEARNINGS.md
```

**Mantra**: "Lo que no está escrito, se olvida."
