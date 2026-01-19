---
description: Desarrollo guiado por tests (TDD workflow)
---

# /tdd - Test-Driven Development

## Uso
```
/tdd <descripción de la funcionalidad>
```

## Proceso

1. **Entender** la funcionalidad a implementar
2. **Escribir test** que define el comportamiento esperado
3. **Ejecutar test** - debe FALLAR
4. **Implementar** código mínimo para pasar
5. **Ejecutar test** - debe PASAR
6. **Refactorizar** si es necesario
7. **Verificar cobertura** >= 80%

## Agente
Delegar a: `.agent/agents/tdd-guide.md`

## Comandos
```bash
# Java/Maven
mvn test -Dtest=NombreTest

# Java/Gradle
./gradlew test --tests NombreTest
```

## Checklist
- [ ] Test escrito ANTES de implementación
- [ ] Nombre descriptivo del test
- [ ] Pattern AAA (Arrange-Act-Assert)
- [ ] Mocks para dependencias externas
- [ ] Cobertura verificada
