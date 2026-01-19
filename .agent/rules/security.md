# Reglas de Seguridad

## Checks Obligatorios

Antes de CUALQUIER commit:

- [ ] **Sin secretos hardcodeados** (API keys, passwords, tokens)
- [ ] **Inputs validados** (todos los datos de usuario)
- [ ] **SQL injection prevenido** (queries parametrizadas)
- [ ] **XSS prevenido** (HTML sanitizado)
- [ ] **CSRF habilitado** (si aplica)
- [ ] **Auth/Authz verificados**
- [ ] **Rate limiting** en endpoints públicos
- [ ] **Logs sin datos sensibles**

## Gestión de Secretos

```java
// ❌ NUNCA
private static final String API_KEY = "sk-proj-xxxxx";

// ✅ SIEMPRE
@Value("${api.key}")
private String apiKey;
```

## Si Encuentras un Problema de Seguridad

1. **STOP** inmediatamente
2. **EVALUAR** severidad
3. **USAR** agente `security-reviewer`
4. **REMEDIAR** antes de continuar
5. **DOCUMENTAR** en LEARNINGS.md
6. **ROTAR** secretos expuestos
7. **BUSCAR** problemas similares en el código

## Red Flags

- Cualquier string que parezca un secreto
- Concatenación de strings en queries SQL
- Input de usuario usado directamente sin validar
- Endpoints sin autenticación que deberían tenerla
- Logs con passwords, tokens, o datos personales
- Dependencias muy desactualizadas
