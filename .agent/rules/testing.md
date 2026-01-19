# Reglas de Testing

## Cobertura Mínima: 80%

## Tipos de Tests (TODOS requeridos)

1. **Unitarios** - Funciones, utilidades, lógica
2. **Integración** - Endpoints, base de datos
3. **E2E** - Flujos críticos de usuario

## TDD Obligatorio

1. Escribir test primero (RED)
2. Ejecutar - debe FALLAR
3. Implementar mínimo (GREEN)
4. Ejecutar - debe PASAR
5. Refactorizar (IMPROVE)
6. Verificar cobertura (80%+)

## Cuando Tests Fallan

1. Usar agente `tdd-guide`
2. Verificar aislamiento de tests
3. Verificar mocks correctos
4. Arreglar implementación, NO el test (a menos que el test esté mal)

## Agentes de Soporte

- **tdd-guide** - Usar PROACTIVAMENTE para nuevas features
- **code-reviewer** - Verificar que tests existen

## Comandos

```bash
# Maven
mvn test
mvn verify  # incluye integration tests

# Gradle
./gradlew test

# Test específico
mvn test -Dtest=UserServiceTest
```
