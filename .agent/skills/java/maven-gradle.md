# Maven y Gradle

## Maven

### Estructura del Proyecto
```
project/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   └── test/
│       ├── java/
│       └── resources/
└── target/
```

### pom.xml Básico
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>myapp</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    
    <properties>
        <java.version>17</java.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Database -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### Comandos Maven
```bash
# Compilar
mvn compile

# Ejecutar tests
mvn test

# Empaquetar (incluye tests)
mvn package

# Instalar en repositorio local
mvn install

# Limpiar + instalar, saltando tests
mvn clean install -DskipTests

# Ejecutar aplicación
mvn spring-boot:run

# Ver árbol de dependencias
mvn dependency:tree

# Buscar dependencias desactualizadas
mvn versions:display-dependency-updates

# Forzar actualización de dependencias
mvn -U clean install

# Ejecutar un test específico
mvn test -Dtest=UserServiceTest

# Ejecutar un método de test
mvn test -Dtest=UserServiceTest#shouldCreateUser

# Generar Javadoc
mvn javadoc:javadoc

# Análisis de código con SpotBugs
mvn spotbugs:check

# Cobertura con JaCoCo
mvn verify
```

---

## Gradle

### Estructura del Proyecto
```
project/
├── build.gradle.kts (o build.gradle)
├── settings.gradle.kts
├── gradle/
│   └── wrapper/
├── gradlew
├── gradlew.bat
├── src/
│   ├── main/
│   └── test/
└── build/
```

### build.gradle.kts (Kotlin DSL)
```kotlin
plugins {
    java
    id("org.springframework.boot") version "3.2.0"
    id("io.spring.dependency-management") version "1.1.4"
}

group = "com.example"
version = "1.0.0-SNAPSHOT"

java {
    sourceCompatibility = JavaVersion.VERSION_17
}

repositories {
    mavenCentral()
}

dependencies {
    // Spring Boot
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    
    // Database
    runtimeOnly("org.postgresql:postgresql")
    
    // Lombok
    compileOnly("org.projectlombok:lombok")
    annotationProcessor("org.projectlombok:lombok")
    
    // Testing
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testImplementation("org.testcontainers:postgresql")
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

### Comandos Gradle
```bash
# Compilar
./gradlew compileJava

# Ejecutar tests
./gradlew test

# Build completo
./gradlew build

# Ejecutar aplicación
./gradlew bootRun

# Limpiar
./gradlew clean

# Ver dependencias
./gradlew dependencies

# Refrescar dependencias
./gradlew --refresh-dependencies build

# Test específico
./gradlew test --tests UserServiceTest

# Método específico
./gradlew test --tests "UserServiceTest.shouldCreateUser"

# Con debug
./gradlew test --debug

# Parallel build
./gradlew build --parallel
```

---

## Gestión de Dependencias

### Buscar Dependencias
- [Maven Central](https://search.maven.org/)
- [MVN Repository](https://mvnrepository.com/)

### Scopes Comunes

| Maven | Gradle | Uso |
|-------|--------|-----|
| compile | implementation | Dependencia normal |
| provided | compileOnly | Provista en runtime (ej: Lombok) |
| runtime | runtimeOnly | Solo en runtime (ej: driver JDBC) |
| test | testImplementation | Solo para tests |

### Excluir Dependencias Transitivas
```xml
<!-- Maven -->
<dependency>
    <groupId>com.example</groupId>
    <artifactId>library</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.unwanted</groupId>
            <artifactId>unwanted-lib</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

```kotlin
// Gradle
implementation("com.example:library") {
    exclude(group = "org.unwanted", module = "unwanted-lib")
}
```

### BOM (Bill of Materials)
```xml
<!-- Maven -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>testcontainers-bom</artifactId>
            <version>1.19.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

```kotlin
// Gradle
dependencyManagement {
    imports {
        mavenBom("org.testcontainers:testcontainers-bom:1.19.0")
    }
}
```

---

## Plugins Útiles

### JaCoCo (Cobertura)
```xml
<!-- Maven -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>verify</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### SpotBugs
```xml
<plugin>
    <groupId>com.github.spotbugs</groupId>
    <artifactId>spotbugs-maven-plugin</artifactId>
    <version>4.8.1.0</version>
</plugin>
```

### Checkstyle
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>3.3.1</version>
</plugin>
```
