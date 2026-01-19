# Debugging Sistemático

## Proceso de 5 Pasos

```
┌─────────────────────────────────────┐
│  1. REPRODUCIR                      │
│  Confirmar que puedo reproducirlo   │
└───────────────┬─────────────────────┘
                │
                ▼
┌─────────────────────────────────────┐
│  2. AISLAR                          │
│  Encontrar el código mínimo         │
└───────────────┬─────────────────────┘
                │
                ▼
┌─────────────────────────────────────┐
│  3. DIAGNOSTICAR                    │
│  Entender la causa raíz             │
└───────────────┬─────────────────────┘
                │
                ▼
┌─────────────────────────────────────┐
│  4. CORREGIR                        │
│  Aplicar fix mínimo                 │
└───────────────┬─────────────────────┘
                │
                ▼
┌─────────────────────────────────────┐
│  5. VERIFICAR                       │
│  Confirmar fix y no regresiones     │
└─────────────────────────────────────┘
```

## 1. REPRODUCIR

### Objetivo
Confirmar que puedo reproducir el error de manera consistente.

### Preguntas
- ¿En qué ambiente ocurre? (dev, staging, prod)
- ¿Es 100% reproducible o intermitente?
- ¿Qué pasos exactos lo provocan?
- ¿Desde cuándo ocurre?
- ¿Qué cambió recientemente?

### Acciones
```bash
# Revisar logs
tail -f logs/app.log | grep ERROR

# Buscar commits recientes
git log --oneline -20

# Ver qué cambió
git diff HEAD~5 HEAD -- src/

# Revisar cambios de deps
git diff HEAD~5 HEAD -- pom.xml package.json
```

## 2. AISLAR

### Objetivo
Reducir el problema al código mínimo necesario.

### Técnicas

#### Binary Search
```
1. ¿El problema está en frontend o backend?
   → Backend
2. ¿En qué capa? Controller, Service, Repository?
   → Service
3. ¿En qué método del Service?
   → createUser()
4. ¿En qué línea del método?
   → Línea 45: userRepository.save()
```

#### Comentar Código
```java
public User createUser(CreateUserRequest req) {
    // validate(req);          // ← ¿falla sin esto?
    // User user = mapper.toEntity(req);
    // userRepository.save(user);
    // emailService.send(user); // ← ¿falla con esto comentado?
    return user;
}
```

#### Minimal Reproducible Example
```java
@Test
void reproducesBug() {
    // Mínimo código que reproduce el error
    var result = service.problematicMethod(input);
    // Falla aquí
}
```

## 3. DIAGNOSTICAR

### Herramientas

#### Logging
```java
log.debug("Input: {}", input);
log.debug("State before: {}", state);
result = problematicMethod(input);
log.debug("State after: {}", state);
log.debug("Output: {}", result);
```

#### Debugger
```
1. Poner breakpoint en la zona sospechosa
2. Ejecutar paso a paso
3. Inspeccionar variables
4. Watch expressions
```

#### Stack Trace
```
- Leer de abajo hacia arriba
- Identificar primera línea de MI código
- Esa es la zona del problema
```

### Preguntas de Diagnóstico

| Síntoma | Posible Causa |
|---------|---------------|
| NullPointerException | Variable no inicializada |
| Resultado incorrecto | Lógica mal implementada |
| Timeout | Loop infinito o N+1 |
| Datos corruptos | Race condition |
| Solo falla en prod | Config diferente |
| Intermitente | Concurrencia o timing |

## 4. CORREGIR

### Principios

1. **Fix mínimo** - Solo lo necesario
2. **Entender antes de cambiar** - No "probar suerte"
3. **Un cambio a la vez** - Para saber qué lo arregló
4. **Escribir test primero** - Que falle, luego arreglar

### Template de Fix
```java
// ANTES (buggy)
public Result process(Input input) {
    return calculate(input.getValue()); // NPE si input.value es null
}

// DESPUÉS (fixed)
public Result process(Input input) {
    if (input == null || input.getValue() == null) {
        throw new IllegalArgumentException("Input and value cannot be null");
    }
    return calculate(input.getValue());
}
```

## 5. VERIFICAR

### Checklist

- [ ] El test que reproduce el bug ahora pasa
- [ ] Todos los tests existentes siguen pasando
- [ ] Probado manualmente el flujo afectado
- [ ] No hay regresiones en funcionalidad relacionada
- [ ] Documentado en LEARNINGS.md si es error nuevo

### Comandos
```bash
# Ejecutar tests
mvn test

# Test específico que reproduce el bug
mvn test -Dtest=UserServiceTest#shouldHandleNullInput

# Verificar que no hay regresiones
mvn verify
```

## Anti-patrones de Debugging

### ❌ Evitar

1. **Cambiar múltiples cosas a la vez**
   - No sabrás qué lo arregló
   
2. **"Funciona en mi máquina"**
   - Comparar configs, versiones, datos

3. **Agregar código sin entender**
   - Los try-catch vacíos no arreglan nada

4. **Debuggear por horas sin descanso**
   - 15 min break puede dar claridad

5. **Asumir que el problema es donde crees**
   - Verifica con datos, no intuición

## Rubber Duck Debugging

Explica el problema en voz alta:

```
"Tengo un servicio que debería retornar usuarios.
Recibe un ID, busca en la base de datos...
Espera, ¿estoy pasando el ID correcto?
*revisa*
¡El ID viene como String y lo comparo con Long!"
```

## Comandos Útiles

```bash
# Ver logs en tiempo real
tail -f logs/app.log

# Buscar errores en logs
grep -i "error\|exception" logs/app.log | tail -50

# Ver procesos
ps aux | grep java

# Ver puertos
lsof -i :8080

# Ver conexiones DB
netstat -an | grep 5432

# Memoria heap
jmap -heap <pid>

# Thread dump
jstack <pid> > threads.txt
```

**Mantra**: "No adivines. Mide. Reproduce. Aísla. Entiende. Arregla."
