---
description: Resolver errores de compilación y build
---

# /build-fix - Arreglar Errores de Build

## Uso
```
/build-fix
```

## Proceso

1. **Ejecutar** build para capturar errores
2. **Analizar** mensaje de error
3. **Categorizar** (syntax, import, dependency, version)
4. **Aplicar** fix apropiado
5. **Verificar** que compila
6. **Documentar** si es error nuevo

## Agente
Delegar a: `.agent/agents/build-error-resolver.md`

## Comandos
```bash
# Maven
mvn clean compile

# Gradle
./gradlew clean compileJava
```

## Errores Comunes

| Error | Causa | Fix |
|-------|-------|-----|
| cannot find symbol | Import faltante | Añadir import |
| package does not exist | Dep faltante | Añadir a pom/gradle |
| incompatible types | Tipo incorrecto | Parsear/cambiar tipo |

## Checklist
- [ ] Error identificado
- [ ] Causa raíz encontrada
- [ ] Fix aplicado
- [ ] Build pasa
- [ ] Documentado si nuevo
