# Estándares de Código Java

## Convenciones de Nomenclatura

### Clases
```java
// PascalCase
public class UserService { }
public class OrderRepository { }
public class PaymentException extends RuntimeException { }
```

### Métodos y Variables
```java
// camelCase
public void createUser(String userName) { }
private int maxRetries = 3;
```

### Constantes
```java
// UPPER_SNAKE_CASE
private static final int MAX_RETRIES = 3;
public static final String DEFAULT_ENCODING = "UTF-8";
```

### Paquetes
```java
// lowercase, dominio inverso
package com.example.service;
package com.example.repository;
```

### Interfaces
```java
// Nombre descriptivo, sin prefijo "I"
public interface UserRepository { }  // ✓
public interface IUserRepository { } // ✗ No usar
```

## Estructura de Clase

```java
public class ExampleService {
    
    // 1. Logger (siempre primero)
    private static final Logger log = LoggerFactory.getLogger(ExampleService.class);
    
    // 2. Constantes estáticas
    private static final int MAX_RETRIES = 3;
    
    // 3. Campos de instancia (preferir final)
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    // 4. Constructor
    public ExampleService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    // 5. Métodos públicos
    public User findById(Long id) { }
    
    // 6. Métodos de paquete (default)
    void internalMethod() { }
    
    // 7. Métodos privados
    private void validateUser(User user) { }
    
    // 8. Clases internas (si las hay)
    private static class UserValidator { }
}
```

## Documentación (Javadoc)

> **IMPORTANTE**: Usar "función" en lugar de "método" en la documentación en español.

### Clase
```java
/**
 * Servicio para gestión de usuarios.
 *
 * <p>Esta función proporciona operaciones CRUD para usuarios
 * incluyendo validación y notificaciones.</p>
 *
 * @author Equipo de Desarrollo
 * @since 1.0
 * @see User
 */
@Service
public class UserService { }
```

### Método Público
```java
/**
 * Busca un usuario por su identificador.
 *
 * @param id Identificador único del usuario
 * @return Usuario encontrado
 * @throws UserNotFoundException si no existe el usuario
 */
public User findById(Long id) { }
```

### No Documentar
- Getters/setters triviales
- Métodos privados obvios
- Constructores simples

## Imports

```java
// ✅ BIEN: Imports explícitos
import java.util.List;
import java.util.Map;
import java.util.Optional;

// ❌ MAL: Wildcard imports
import java.util.*;

// ❌ MAL: Nombre completo en código
public java.util.List<String> getNames() { }
```

### Orden de Imports
1. java.*
2. javax.*
3. org.*
4. com.*
5. static imports (al final)

## Mejores Prácticas

### Inmutabilidad
```java
// Preferir campos final
private final String name;

// Usar records para DTOs
public record CreateUserRequest(String email, String password) { }

// Devolver copias defensivas
public List<User> getUsers() {
    return List.copyOf(this.users);
}
```

### Optional
```java
// ✅ Retorno en métodos que pueden no tener valor
public Optional<User> findByEmail(String email);

// ❌ No usar en parámetros
public void process(Optional<User> user); // MAL

// ❌ No usar en campos
private Optional<User> currentUser; // MAL
```

### Manejo de Null
```java
// Validar parámetros de entrada
public void setName(String name) {
    this.name = Objects.requireNonNull(name, "name cannot be null");
}

// Usar Optional para retornos
public Optional<User> findById(Long id) {
    return Optional.ofNullable(userRepository.findById(id));
}
```

### Streams
```java
// Preferir streams para transformaciones
List<String> names = users.stream()
    .filter(User::isActive)
    .map(User::getName)
    .sorted()
    .collect(Collectors.toList());

// Usar toList() en Java 16+
List<String> names = users.stream()
    .map(User::getName)
    .toList();
```

### Try-with-resources
```java
// ✅ Siempre para recursos
try (InputStream is = new FileInputStream(file);
     BufferedReader reader = new BufferedReader(new InputStreamReader(is))) {
    // usar
}
```

## Límites de Tamaño

| Elemento | Límite | Acción si se excede |
|----------|--------|---------------------|
| Método | 50 líneas | Extraer métodos privados |
| Clase | 400 líneas | Dividir responsabilidades |
| Archivo | 800 líneas | Refactorizar urgente |
| Parámetros | 4 | Usar objeto/record |
| Anidamiento | 4 niveles | Guard clauses |

## Anti-patrones

### Evitar
- Números mágicos (usar constantes)
- Strings hardcodeados repetidos
- Catch de Exception genérico
- System.out.println (usar logger)
- Campos públicos
- Getters que hacen lógica
- Static para todo

### Preferir
- Constantes con nombre descriptivo
- Enums para valores fijos
- Catch específico + log
- Logger con nivel apropiado
- Encapsulación
- Métodos de negocio claros
- Inyección de dependencias
