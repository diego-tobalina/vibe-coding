---
name: java-testing
description: Java testing guidelines with JUnit 5, Mockito, AssertJ, and Testcontainers. Use when writing tests for Java code.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Edit
  - Write
---

# Java Testing Guidelines

## Dependencies

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
```

## Test Structure

```
src/test/java/com/company/project/
└── user/
    ├── UserServiceTest.java           # Unit tests
    ├── UserServiceIntegrationTest.java
    └── UserRepositoryTest.java
```

## Naming Convention

```
methodName_scenarioDescription_expectedOutcome
```

## Unit Test Setup

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private UserMapper userMapper;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void create_withValidData_returnsCreatedUser() {
        // Given
        UserCreateDto dto = UserCreateDto.builder()
            .email("test@example.com")
            .name("Test User")
            .build();
        User user = User.builder().id(UUID.randomUUID()).build();
        
        when(userRepository.existsByEmail(anyString())).thenReturn(false);
        when(userMapper.toEntity(any(UserCreateDto.class))).thenReturn(user);
        when(userRepository.save(any(User.class))).thenReturn(user);
        
        // When
        UserDto result = userService.create(dto);
        
        // Then
        assertThat(result).isNotNull();
        verify(userRepository).save(any(User.class));
    }
    
    @Test
    void findById_nonExisting_throwsException() {
        // Given
        UUID id = UUID.randomUUID();
        when(userRepository.findById(id)).thenReturn(Optional.empty());
        
        // When/Then
        assertThatThrownBy(() -> userService.findById(id))
            .isInstanceOf(UserNotFoundException.class);
    }
}
```

## Integration Test with Testcontainers

```java
@SpringBootTest
@Testcontainers
class UserServiceIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Autowired
    private UserService userService;
    
    @Test
    void create_persistsToDatabase() {
        UserCreateDto dto = UserCreateDto.builder()
            .email("test@example.com")
            .name("Test")
            .build();
        
        UserDto result = userService.create(dto);
        
        assertThat(result.getId()).isNotNull();
    }
}
```

## Controller Test

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void create_withValidData_returns201() throws Exception {
        UserDto response = UserDto.builder().id("123").build();
        when(userService.create(any())).thenReturn(response);
        
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"email": "test@example.com", "name": "Test"}
                    """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value("123"));
    }
}
```

## Test Data Builder

```java
public class UserTestBuilder {
    public static User.UserBuilder aUser() {
        return User.builder()
            .id(UUID.randomUUID())
            .email("default@example.com")
            .name("Default User")
            .active(true);
    }
}
```

## Repository Test with @DataJpaTest

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.NONE)
@Testcontainers
class UserRepositoryTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void findByEmail_existingUser_returnsUser() {
        // Given
        User user = User.builder().email("test@example.com").name("Test").build();
        userRepository.save(user);
        
        // When
        Optional<User> result = userRepository.findByEmail("test@example.com");
        
        // Then
        assertThat(result).isPresent();
        assertThat(result.get().getName()).isEqualTo("Test");
    }
}
```

## Coverage Goals

- Unit tests: 80% minimum
- Critical business logic: 100%
- Integration tests for all repositories and controllers

## Commands

```bash
# Run all tests
mvn test

# Run specific test class
mvn test -Dtest=UserServiceTest

# Run specific test method
mvn test -Dtest=UserServiceTest#create_withValidData_returnsCreatedUser

# Run with coverage
mvn test jacoco:report
```
