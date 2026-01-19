# Lombok y MapStruct

## Lombok

### Dependencias

```xml
<!-- Maven -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

```kotlin
// Gradle
compileOnly("org.projectlombok:lombok")
annotationProcessor("org.projectlombok:lombok")
```

### Anotaciones Principales

| Anotación | Genera |
|-----------|--------|
| `@Getter` | Getters para todos los campos |
| `@Setter` | Setters para todos los campos |
| `@NoArgsConstructor` | Constructor sin argumentos |
| `@AllArgsConstructor` | Constructor con todos los argumentos |
| `@RequiredArgsConstructor` | Constructor para campos final |
| `@Builder` | Patrón Builder |
| `@Data` | @Getter + @Setter + @ToString + @EqualsAndHashCode |
| `@Value` | Clase inmutable (@Data + final) |
| `@Slf4j` | Logger SLF4J |
| `@ToString` | Método toString() |
| `@EqualsAndHashCode` | equals() y hashCode() |

### Patrones Recomendados

#### Entity JPA
```java
@Entity
@Table(name = "users")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder(toBuilder = true)
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String email;
    private String name;
    
    // Para JPA: equals/hashCode basado en ID
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User user)) return false;
        return id != null && id.equals(user.id);
    }
    
    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}
```

#### Service
```java
@Service
@RequiredArgsConstructor
@Slf4j
public class UserService {
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    
    public UserResponse create(CreateUserRequest request) {
        log.info("Creating user: {}", request.email());
        User user = userMapper.toEntity(request);
        User saved = userRepository.save(user);
        return userMapper.toResponse(saved);
    }
}
```

#### Controller
```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
@Slf4j
public class UserController {
    private final UserService userService;
    // ...
}
```

#### DTO Inmutable
```java
@Value
@Builder
public class UserResponse {
    Long id;
    String email;
    String name;
}
```

### Lombok Config

Crear `lombok.config` en raíz del proyecto:
```properties
# Añadir @Generated a métodos generados
lombok.addLombokGeneratedAnnotation = true

# Deshabilitar @Setter para entidades (inmutabilidad)
# lombok.setter.flagUsage = warning

# Usar constructores con parámetros nombrados
lombok.anyConstructor.addConstructorProperties = true
```

---

## MapStruct

### Dependencias

```xml
<!-- Maven -->
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.5.Final</version>
</dependency>

<!-- Annotation processor -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <annotationProcessorPaths>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>1.5.5.Final</version>
            </path>
            <path>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
            </path>
            <path>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok-mapstruct-binding</artifactId>
                <version>0.2.0</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

```kotlin
// Gradle
implementation("org.mapstruct:mapstruct:1.5.5.Final")
annotationProcessor("org.mapstruct:mapstruct-processor:1.5.5.Final")
annotationProcessor("org.projectlombok:lombok-mapstruct-binding:0.2.0")
```

### Mapper Básico

```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    
    UserResponse toResponse(User user);
    
    List<UserResponse> toResponseList(List<User> users);
    
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "status", constant = "ACTIVE")
    User toEntity(CreateUserRequest request);
}
```

### Mapeos Avanzados

```java
@Mapper(componentModel = "spring", uses = {AddressMapper.class})
public interface UserMapper {
    
    // Campo diferente nombre
    @Mapping(source = "userName", target = "name")
    UserResponse toResponse(User user);
    
    // Campo anidado
    @Mapping(source = "address.city", target = "city")
    UserResponse toResponseWithCity(User user);
    
    // Expresión Java
    @Mapping(target = "fullName", expression = "java(user.getFirstName() + \" \" + user.getLastName())")
    UserResponse toResponseWithFullName(User user);
    
    // Valor por defecto
    @Mapping(target = "status", defaultValue = "UNKNOWN")
    UserResponse toResponseWithDefault(User user);
    
    // Ignorar campo
    @Mapping(target = "password", ignore = true)
    UserResponse toResponseSafe(User user);
    
    // Actualizar entidad existente
    @Mapping(target = "id", ignore = true)
    void updateEntity(UpdateUserRequest request, @MappingTarget User user);
}
```

### Mapper con Dependencias

```java
@Mapper(componentModel = "spring")
public abstract class UserMapper {
    
    @Autowired
    protected PasswordEncoder passwordEncoder;
    
    @Mapping(target = "password", expression = "java(passwordEncoder.encode(request.password()))")
    public abstract User toEntity(CreateUserRequest request);
}
```

### Configuración Global

```java
@MapperConfig(
    componentModel = "spring",
    nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE,
    nullValueCheckStrategy = NullValueCheckStrategy.ALWAYS
)
public interface MapperConfiguration {
}

// Usar en mappers
@Mapper(config = MapperConfiguration.class)
public interface UserMapper {
    // ...
}
```

---

## Uso Combinado

### Flujo Completo

```java
// 1. Entity con Lombok
@Entity
@Getter @Setter @NoArgsConstructor @AllArgsConstructor @Builder
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String email;
    private String name;
}

// 2. DTOs con Lombok
@Value @Builder
public class CreateUserRequest {
    @NotBlank @Email String email;
    @NotBlank String name;
}

@Value @Builder  
public class UserResponse {
    Long id;
    String email;
    String name;
}

// 3. Mapper con MapStruct
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserResponse toResponse(User user);
    
    @Mapping(target = "id", ignore = true)
    User toEntity(CreateUserRequest request);
    
    void updateEntity(UpdateUserRequest request, @MappingTarget User user);
}

// 4. Service usando todo
@Service
@RequiredArgsConstructor
@Slf4j
public class UserService {
    private final UserRepository repository;
    private final UserMapper mapper;
    
    public UserResponse create(CreateUserRequest request) {
        log.info("Creating user: {}", request.getEmail());
        User entity = mapper.toEntity(request);
        User saved = repository.save(entity);
        return mapper.toResponse(saved);
    }
    
    public UserResponse update(Long id, UpdateUserRequest request) {
        User entity = repository.findById(id)
            .orElseThrow(() -> new EntityNotFoundException("User not found"));
        mapper.updateEntity(request, entity);
        return mapper.toResponse(repository.save(entity));
    }
}
```

## Mejores Prácticas

1. **Usar `@RequiredArgsConstructor`** para inyección por constructor
2. **Usar `@Slf4j`** en lugar de crear Logger manual
3. **Usar `@Builder(toBuilder = true)`** para copias modificables
4. **MapStruct para TODOS los mapeos** Entity ↔ DTO
5. **Nunca exponer entidades** en APIs, siempre usar DTOs
6. **Combinar Lombok + MapStruct** para código mínimo
