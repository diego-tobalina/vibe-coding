# TDD Workflow

## El Ciclo TDD

```
    ┌─────────┐
    │   RED   │ ← Escribir test que FALLA
    └────┬────┘
         │
         ▼
    ┌─────────┐
    │  GREEN  │ ← Código MÍNIMO para pasar
    └────┬────┘
         │
         ▼
    ┌─────────┐
    │REFACTOR │ ← Mejorar sin cambiar comportamiento
    └────┬────┘
         │
         └───────→ Repetir
```

## Paso a Paso

### 1. RED - Escribir Test
```java
@Test
void shouldCalculateTotalWithDiscount() {
    // Arrange
    Order order = new Order();
    order.addItem(new Item("A", 100.0));
    order.addItem(new Item("B", 50.0));
    order.setDiscount(0.1); // 10%
    
    // Act
    double total = order.calculateTotal();
    
    // Assert
    assertThat(total).isEqualTo(135.0);
}
```

### 2. Ejecutar - Verificar que FALLA
```bash
mvn test -Dtest=OrderTest#shouldCalculateTotalWithDiscount
```
El test DEBE fallar. Si pasa sin implementación, está mal escrito.

### 3. GREEN - Implementación Mínima
```java
public double calculateTotal() {
    double subtotal = items.stream()
        .mapToDouble(Item::getPrice)
        .sum();
    return subtotal * (1 - discount);
}
```

### 4. Ejecutar - Verificar que PASA
```bash
mvn test -Dtest=OrderTest#shouldCalculateTotalWithDiscount
```

### 5. REFACTOR - Mejorar
- Extraer métodos
- Mejorar nombres
- Eliminar duplicación
- **Sin cambiar comportamiento**

### 6. Ejecutar todos los tests
```bash
mvn test
```

## Reglas de TDD

1. **No escribir código de producción sin un test que falle**
2. **Escribir solo el test suficiente para fallar**
3. **Escribir solo el código suficiente para pasar**

## Beneficios

- Diseño emergente
- Cobertura alta automáticamente
- Documentación viva
- Refactoring seguro
- Menos bugs
- Desarrollo más rápido a largo plazo

## Cobertura Objetivo

- **Global:** 80% mínimo
- **Clases de servicio:** 90%
- **Código crítico:** 95%

## Tipos de Tests

| Tipo | Qué testea | Velocidad | Aislamiento |
|------|-----------|-----------|-------------|
| Unitario | Una función/clase | Muy rápido | Total (mocks) |
| Integración | Varios componentes | Medio | Parcial |
| E2E | Sistema completo | Lento | Ninguno |

## Pirámide de Tests

```
        /\
       /E2E\        ← Pocos, lentos, costosos
      /──────\
     /Integr.\     ← Algunos, velocidad media
    /──────────\
   /  Unitarios \  ← Muchos, rápidos, baratos
  ────────────────
```
