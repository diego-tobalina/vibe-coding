# Principios de Código Limpio

## Principios Fundamentales

### SOLID

1. **S**ingle Responsibility - Una clase, una responsabilidad
2. **O**pen/Closed - Abierto a extensión, cerrado a modificación
3. **L**iskov Substitution - Subtipos sustituibles por tipos base
4. **I**nterface Segregation - Interfaces pequeñas y específicas
5. **D**ependency Inversion - Depender de abstracciones

### KISS
Keep It Simple, Stupid - La solución más simple suele ser la mejor.

### DRY
Don't Repeat Yourself - Evitar duplicación de lógica.

### YAGNI
You Aren't Gonna Need It - No implementar funcionalidad hasta que sea necesaria.

## Nomenclatura

### Nombres Descriptivos
```
// ❌ Mal
int d; // días transcurridos
List<int> list1;
void calc();

// ✅ Bien
int daysSinceCreation;
List<int> activeUserIds;
void calculateMonthlyRevenue();
```

### Convenciones
- **Clases/Interfaces**: PascalCase
- **Métodos/Variables**: camelCase
- **Constantes**: UPPER_SNAKE_CASE
- **Booleanos**: isX, hasX, canX, shouldX

## Funciones

### Pequeñas y Enfocadas
```
// ❌ Mal: Función que hace demasiado
void processUserRegistration(...) {
  // validar datos (20 líneas)
  // guardar en BD (15 líneas)
  // enviar email (10 líneas)
  // actualizar estadísticas (10 líneas)
}

// ✅ Bien: Funciones pequeñas
void processUserRegistration(...) {
  validateData(data);
  User user = saveUser(data);
  sendWelcomeEmail(user);
  updateStatistics(user);
}
```

### Parámetros
- Máximo 3-4 parámetros
- Usar objetos para más parámetros
- Evitar parámetros booleanos (extraer a dos métodos)

## Comentarios

### Cuándo Comentar
- Explicar el **porqué**, no el **qué**
- Documentar APIs públicas
- Aclarar decisiones no obvias
- Avisar de consecuencias inesperadas

### Cuándo NO Comentar
- Código obvio
- Código que debería ser refactorizado
- Histórico (usar Git)

```
// ❌ Mal
i++; // incrementar i

// ✅ Bien
// Cache agresivo porque la API externa tiene límite de 100 req/min
cache.setTtl(60);
```

## Manejo de Errores

### Usar Excepciones, No Códigos de Error
```
// ❌ Mal
int result = process(data);
if (result == -1) { /* error */ }

// ✅ Bien
try {
  process(data);
} catch (ProcessingException e) {
  // manejar
}
```

### No Ignorar Excepciones
```
// ❌ Mal
try {
  ...
} catch (Exception e) {
  // ignorar
}

// ✅ Bien
try {
  ...
} catch (Exception e) {
  log.error("Failed to process", e);
  throw new ProcessingException("...", e);
}
```

## Formato

### Consistencia
- Usar linter/formatter automático
- Seguir estilo del proyecto
- Indentación consistente

### Organización Vertical
- Variables cerca de su uso
- Funciones llamantes arriba de llamadas
- Conceptos relacionados juntos

## Tests

### Características de Buenos Tests
- **F**ast - Rápidos
- **I**ndependent - Independientes entre sí
- **R**epeatable - Mismo resultado siempre
- **S**elf-validating - Pass/fail claro
- **T**imely - Escritos a tiempo (TDD)

## Refactoring

### Señales de que es Necesario
- Código duplicado
- Funciones largas
- Muchos parámetros
- Comentarios excesivos explicando el código
- Feature envy (clase usa más datos de otra)
- Data clumps (datos que siempre van juntos)

### Proceso Seguro
1. Tener tests que pasen
2. Hacer cambio pequeño
3. Ejecutar tests
4. Repetir
