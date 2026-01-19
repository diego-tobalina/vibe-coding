# Testing en Java

## Stack de Testing

- **JUnit 5**: Framework de testing
- **Mockito**: Mocking
- **AssertJ**: Assertions fluidas
- **Spring Test**: Tests de integración
- **Testcontainers**: Contenedores para tests

## Test Unitario Básico

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import static org.assertj.core.api.Assertions.*;

class CalculatorTest {
    
    private Calculator calculator;
    
    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }
    
    @Test
    @DisplayName("should add two positive numbers")
    void shouldAddPositiveNumbers() {
        // Arrange
        int a = 5;
        int b = 3;
        
        // Act
        int result = calculator.add(a, b);
        
        // Assert
        assertThat(result).isEqualTo(8);
    }
    
    @Test
    @DisplayName("should throw when dividing by zero")
    void shouldThrowOnDivisionByZero() {
        // Act & Assert
        assertThatThrownBy(() -> calculator.divide(10, 0))
            .isInstanceOf(ArithmeticException.class)
            .hasMessage("Cannot divide by zero");
    }
}
```

## Test con Mockito

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.mockito.Mockito.*;
import static org.assertj.core.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void shouldReturnUserWhenFound() {
        // Arrange
        Long userId = 1L;
        User expected = new User(userId, "test@example.com");
        when(userRepository.findById(userId)).thenReturn(Optional.of(expected));
        
        // Act
        User result = userService.findById(userId);
        
        // Assert
        assertThat(result).isEqualTo(expected);
        verify(userRepository).findById(userId);
        verifyNoMoreInteractions(userRepository, emailService);
    }
    
    @Test
    void shouldThrowWhenUserNotFound() {
        // Arrange
        Long userId = 999L;
        when(userRepository.findById(userId)).thenReturn(Optional.empty());
        
        // Act & Assert
        assertThatThrownBy(() -> userService.findById(userId))
            .isInstanceOf(UserNotFoundException.class)
            .hasMessageContaining("999");
    }
    
    @Test
    void shouldCreateUserAndSendEmail() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("test@example.com", "password");
        User savedUser = new User(1L, "test@example.com");
        
        when(userRepository.existsByEmail(anyString())).thenReturn(false);
        when(userRepository.save(any(User.class))).thenReturn(savedUser);
        
        // Act
        User result = userService.create(request);
        
        // Assert
        assertThat(result.getId()).isEqualTo(1L);
        verify(userRepository).save(any(User.class));
        verify(emailService).sendWelcome(savedUser);
    }
}
```

## Test de Integración con Spring

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest
@AutoConfigureMockMvc
class UserControllerIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private UserRepository userRepository;
    
    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }
    
    @Test
    void shouldCreateUser() throws Exception {
        // Arrange
        String requestBody = """
            {
                "email": "test@example.com",
                "password": "password123",
                "name": "Test User"
            }
            """;
        
        // Act & Assert
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestBody))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").exists())
            .andExpect(jsonPath("$.email").value("test@example.com"))
            .andExpect(jsonPath("$.name").value("Test User"));
    }
    
    @Test
    void shouldReturn404WhenUserNotFound() throws Exception {
        mockMvc.perform(get("/api/v1/users/999"))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.code").value("NOT_FOUND"));
    }
    
    @Test
    void shouldReturnValidationError() throws Exception {
        String requestBody = """
            {
                "email": "invalid-email",
                "password": "short"
            }
            """;
        
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestBody))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.code").value("VALIDATION_ERROR"));
    }
}
```

## Test con Testcontainers

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@SpringBootTest
@Testcontainers
class UserRepositoryTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldSaveAndFindUser() {
        // Arrange
        User user = User.builder()
            .email("test@example.com")
            .password("password")
            .name("Test")
            .status(UserStatus.ACTIVE)
            .build();
        
        // Act
        User saved = userRepository.save(user);
        Optional<User> found = userRepository.findById(saved.getId());
        
        // Assert
        assertThat(found).isPresent();
        assertThat(found.get().getEmail()).isEqualTo("test@example.com");
    }
}
```

## Patrones de Testing

### AAA (Arrange-Act-Assert)
```java
@Test
void shouldDoSomething() {
    // Arrange - preparar datos y mocks
    
    // Act - ejecutar la acción
    
    // Assert - verificar resultados
}
```

### Test Data Builder
```java
public class UserBuilder {
    private Long id = 1L;
    private String email = "default@example.com";
    private String name = "Default User";
    
    public UserBuilder withId(Long id) {
        this.id = id;
        return this;
    }
    
    public UserBuilder withEmail(String email) {
        this.email = email;
        return this;
    }
    
    public User build() {
        return new User(id, email, name);
    }
}

// Uso
User user = new UserBuilder()
    .withEmail("custom@example.com")
    .build();
```

## AssertJ Assertions

```java
// Básicas
assertThat(result).isEqualTo(expected);
assertThat(result).isNotNull();
assertThat(result).isNull();

// Strings
assertThat(name).startsWith("John");
assertThat(name).contains("Doe");
assertThat(name).matches("^[A-Z].*");

// Números
assertThat(age).isGreaterThan(18);
assertThat(price).isBetween(10.0, 100.0);
assertThat(count).isZero();

// Colecciones
assertThat(users).hasSize(3);
assertThat(users).contains(user1, user2);
assertThat(users).extracting("email").contains("a@b.com", "c@d.com");
assertThat(users).isEmpty();

// Excepciones
assertThatThrownBy(() -> service.fail())
    .isInstanceOf(IllegalArgumentException.class)
    .hasMessage("Invalid input");

// Optional
assertThat(optional).isPresent();
assertThat(optional).isEmpty();
assertThat(optional).hasValue(expected);
```

## Mockito Matchers

```java
// Any
when(repo.findById(anyLong())).thenReturn(Optional.of(user));
when(repo.save(any(User.class))).thenReturn(user);

// Argument Captor
ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);
verify(repo).save(captor.capture());
User captured = captor.getValue();
assertThat(captured.getEmail()).isEqualTo("test@example.com");

// Verify orden
InOrder inOrder = inOrder(repo, emailService);
inOrder.verify(repo).save(any());
inOrder.verify(emailService).send(any());

// Verify times
verify(repo, times(1)).save(any());
verify(repo, never()).delete(any());
verify(repo, atLeastOnce()).findById(any());
```

## Configuración JUnit 5

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)  // Compartir estado entre tests
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)  // Ordenar tests
class OrderedTest {
    
    @Test
    @Order(1)
    void firstTest() { }
    
    @Test
    @Order(2)
    void secondTest() { }
}

// Tests parametrizados
@ParameterizedTest
@ValueSource(strings = {"", " ", "   "})
void shouldRejectBlankNames(String name) {
    assertThatThrownBy(() -> user.setName(name))
        .isInstanceOf(IllegalArgumentException.class);
}

@ParameterizedTest
@CsvSource({
    "1, 1, 2",
    "2, 3, 5",
    "10, -5, 5"
})
void shouldAddNumbers(int a, int b, int expected) {
    assertThat(calculator.add(a, b)).isEqualTo(expected);
}
```

## Cobertura con JaCoCo

```bash
# Ejecutar tests con cobertura
mvn verify

# Ver reporte
open target/site/jacoco/index.html
```

Objetivo mínimo: **80%** de cobertura de líneas.
