# Reglas Java

## Documentación

> **IMPORTANTE**: Usar "función" en lugar de "método" en documentación española.

```java
/**
 * Calcula el precio total.
 *
 * Esta función multiplica cantidad por precio unitario.
 *
 * @param cantidad Número de items
 * @param precio Precio por unidad
 * @return Precio total calculado
 */
public double calcularPrecio(int cantidad, double precio) { }
```

## Imports

Usar imports explícitos, NO wildcards:

```java
// ✅ BIEN
import java.util.List;
import java.util.Map;

// ❌ MAL
import java.util.*;
```

## Spring

- Inyección por **constructor** (no @Autowired en campos)
- @Transactional en métodos que modifican BD
- @Valid en @RequestBody
- Logger en lugar de System.out
- @ExceptionHandler para manejo de errores

## Lombok (OBLIGATORIO)

- `@RequiredArgsConstructor` para inyección (NO @Autowired)
- `@Slf4j` para logging (NO crear Logger manual)
- `@Getter/@Setter` para entidades JPA
- `@Builder(toBuilder = true)` para crear copias
- `@Value` para DTOs inmutables

```java
// ✅ BIEN
@Service
@RequiredArgsConstructor
@Slf4j
public class UserService {
    private final UserRepository repo;
}

// ❌ MAL
@Service
public class UserService {
    private static final Logger log = LoggerFactory.getLogger(UserService.class);
    
    @Autowired
    private UserRepository repo;
}
```

## MapStruct (OBLIGATORIO)

- Usar para TODOS los mapeos Entity ↔ DTO
- Configurar `componentModel = "spring"`
- `@Mapping(target = "x", ignore = true)` para campos no mapeados
- `@MappingTarget` para actualizar entidades existentes

```java
// ✅ BIEN
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserResponse toResponse(User user);
    User toEntity(CreateUserRequest request);
}

// ❌ MAL - Mapeo manual
public static UserResponse from(User user) {
    return new UserResponse(user.getId(), user.getEmail());
}
```

## JPA

- equals/hashCode basados en ID
- FetchType.LAZY por defecto
- JOIN FETCH para evitar N+1
- Índices en campos buscados

## Opcionales

- Optional para retornos que pueden no tener valor
- NO usar Optional en parámetros
- NO usar Optional en campos

## Nombres de Paquetes

```
com.example.proyecto
├── controller    # REST controllers
├── service       # Lógica de negocio
├── repository    # Acceso a datos
├── entity        # Entidades JPA
├── dto           # Request/Response objects
├── exception     # Excepciones custom
├── config        # Configuración
└── util          # Utilidades
```
