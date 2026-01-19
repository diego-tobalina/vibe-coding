# Estilo de Código

## Inmutabilidad (CRÍTICO)

SIEMPRE crear nuevos objetos, NUNCA mutar:

```java
// ❌ MAL: Mutación
public void updateUser(User user, String name) {
    user.setName(name); // MUTACIÓN!
}

// ✅ BIEN: Inmutabilidad (records, builders)
public User updateUser(User user, String name) {
    return user.toBuilder()
        .name(name)
        .build();
}
```

## Organización de Archivos

MUCHOS ARCHIVOS PEQUEÑOS > POCOS ARCHIVOS GRANDES:

| Límite | Valor | Acción |
|--------|-------|--------|
| Método | 50 líneas | Extraer métodos |
| Clase | 400 líneas | Dividir responsabilidades |
| Archivo | 800 líneas | Refactor urgente |

## Manejo de Errores

SIEMPRE manejar errores comprehensivamente:

```java
try {
    Result result = riskyOperation();
    return result;
} catch (SpecificException e) {
    log.error("Operation failed: {}", e.getMessage(), e);
    throw new BusinessException("User-friendly message", e);
}
```

## Validación de Input

SIEMPRE validar entrada de usuario:

```java
// Con Bean Validation
public record CreateUserRequest(
    @NotBlank @Email String email,
    @NotBlank @Size(min = 8) String password
) { }

// En controller
public ResponseEntity<?> create(@Valid @RequestBody CreateUserRequest req) { }
```

## Checklist Pre-Commit

- [ ] Código legible y bien nombrado
- [ ] Funciones pequeñas (<50 líneas)
- [ ] Sin anidamiento profundo (>4 niveles)
- [ ] Errores manejados apropiadamente
- [ ] Sin System.out.println
- [ ] Sin valores hardcodeados
- [ ] Patrones inmutables usados
- [ ] Imports explícitos (no wildcards)
