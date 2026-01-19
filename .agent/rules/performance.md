# Reglas de Performance

## Gestión de Contexto

- Mantener archivos de memoria actualizados
- Limpiar contexto innecesario periódicamente
- Priorizar información relevante para la tarea actual

## Base de Datos

- Usar paginación para listas grandes
- Evitar N+1 queries (JOIN FETCH)
- Indexar campos frecuentemente buscados
- Limitar resultados de queries

```java
// ❌ MAL: Todos los registros
List<User> users = userRepository.findAll();

// ✅ BIEN: Paginado
Page<User> users = userRepository.findAll(PageRequest.of(0, 20));
```

## Caching

- Cachear datos que cambian poco
- TTL apropiado según el dato
- Invalidar cache cuando cambian datos

```java
@Cacheable("users")
public User findById(Long id) { }

@CacheEvict(value = "users", key = "#user.id")
public User update(User user) { }
```

## Async

- Operaciones I/O intensivas en async
- No bloquear thread principal
- Timeouts en llamadas externas

```java
@Async
public CompletableFuture<Report> generateReport() { }

// Con timeout
restTemplate.setReadTimeout(5000);
```

## Logging

- Nivel apropiado (DEBUG, INFO, WARN, ERROR)
- No loggear en loops calientes
- Incluir contexto relevante

```java
// ✅ BIEN
log.info("Created user: id={}", user.getId());

// ❌ MAL
log.info("Created user: " + user.toString()); // String concat
```
