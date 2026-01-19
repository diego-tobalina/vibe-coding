# Diseño de APIs REST

## Principios

### Recursos como Sustantivos
```
✅ GET /users
✅ GET /users/123
✅ POST /users
✅ GET /users/123/orders

❌ GET /getUsers
❌ POST /createUser
❌ GET /getUserOrders
```

### Métodos HTTP
| Método | Uso | Idempotente |
|--------|-----|-------------|
| GET | Obtener recursos | Sí |
| POST | Crear recurso | No |
| PUT | Reemplazar recurso | Sí |
| PATCH | Actualizar parcial | No |
| DELETE | Eliminar recurso | Sí |

### Códigos de Estado
| Código | Significado | Uso |
|--------|-------------|-----|
| 200 | OK | GET, PUT, PATCH exitosos |
| 201 | Created | POST exitoso |
| 204 | No Content | DELETE exitoso |
| 400 | Bad Request | Datos inválidos |
| 401 | Unauthorized | Sin autenticación |
| 403 | Forbidden | Sin autorización |
| 404 | Not Found | Recurso no existe |
| 409 | Conflict | Conflicto (ej: duplicado) |
| 500 | Internal Error | Error del servidor |

## Estructura de Respuesta

### Éxito
```json
{
  "data": {
    "id": 123,
    "name": "John",
    "email": "john@example.com"
  }
}
```

### Lista con Paginación
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 150,
    "totalPages": 8
  }
}
```

### Error
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email format is invalid",
    "details": [
      { "field": "email", "message": "must be valid email" }
    ]
  }
}
```

## Versionado

```
/api/v1/users
/api/v2/users
```

## Query Parameters

### Filtrado
```
GET /users?status=active&role=admin
```

### Paginación
```
GET /users?page=2&size=20
```

### Ordenamiento
```
GET /users?sort=createdAt&order=desc
```

### Búsqueda
```
GET /users?q=john
```

## Best Practices

1. Usar HTTPS siempre
2. Validar todos los inputs
3. Paginar colecciones grandes
4. Documentar con OpenAPI/Swagger
5. Rate limiting
6. Versionar la API
7. Respuestas consistentes
8. HATEOAS para discoverabilidad (opcional)
