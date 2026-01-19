# 🤖 Agent Configuration System

Sistema modular de configuración para agentes de IA de código (Claude Code, Gemini CLI, Cursor, Copilot, etc.). Inspirado en [everything-claude-code](https://github.com/affaan-m/everything-claude-code) y [antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills).

## ✨ Características

- 🔄 **Ejecución persistente estilo RALPH** - Auto-recuperación y aprendizaje continuo
- ☕ **Orientado a Java/Spring Boot** - Skills y reglas específicas
- 📝 **Sistema de memoria** - CONTEXT, LEARNINGS, QUEUE, LOGS
- 🧩 **13 agentes especializados** - Planificación, revisión, debugging, etc.
- 📚 **15 skills** - Patrones reutilizables por lenguaje
- ⚡ **7 workflows** - Slash commands para tareas comunes

## 📁 Estructura

```
.agent/
├── AGENTS.md              # Sistema principal (RALPH-style)
├── CONTEXT.md             # Mapa del proyecto
├── LEARNINGS.md           # Base de conocimiento
├── QUEUE.md               # Estado de tareas
├── LOGS.md                # Registro de ejecución
├── tasks/                 # Cola de tareas
├── agents/                # 13 agentes especializados
│   ├── planner.md
│   ├── architect.md
│   ├── java-specialist.md
│   ├── tdd-guide.md
│   ├── code-reviewer.md
│   ├── security-reviewer.md
│   ├── build-error-resolver.md
│   ├── refactor-cleaner.md
│   ├── doc-updater.md
│   ├── task-generator.md
│   ├── react-specialist.md
│   ├── knowledge-updater.md
│   └── e2e-runner.md
├── skills/
│   ├── java/              # 6 skills Java
│   │   ├── coding-standards.md
│   │   ├── spring-patterns.md
│   │   ├── maven-gradle.md
│   │   ├── testing-java.md
│   │   ├── lombok-mapstruct.md
│   │   └── redis-caching.md
│   ├── frontend/          # 2 skills Frontend
│   │   ├── react-patterns.md
│   │   └── typescript-standards.md
│   └── general/           # 7 skills generales
│       ├── clean-code.md
│       ├── git-workflow.md
│       ├── api-design.md
│       ├── tdd-workflow.md
│       ├── systematic-debugging.md
│       ├── github-actions.md
│       └── conversation-memory.md
├── rules/                 # 5 reglas obligatorias
│   ├── security.md
│   ├── coding-style.md
│   ├── testing.md
│   ├── java-rules.md
│   └── performance.md
├── workflows/             # 7 slash commands
│   ├── tdd.md
│   ├── plan.md
│   ├── code-review.md
│   ├── build-fix.md
│   ├── refactor.md
│   ├── generate-task.md
│   └── add-knowledge.md
└── templates/             # Plantillas
    ├── task.md
    ├── adr.md
    └── learning.md
```

## 🚀 Instalación

```bash
# Clonar en tu proyecto
git clone https://github.com/tu-usuario/agent-config.git .agent

# O copiar solo la carpeta .agent a tu proyecto existente
cp -r agent-config/.agent /tu/proyecto/
```

## 🔧 Compatibilidad

| Herramienta | Directorio |
|-------------|-----------|
| Claude Code | `.claude/` o `.agent/` |
| Gemini CLI | `.gemini/` o `.agent/` |
| Cursor | `.cursor/` o `.agent/` |
| Copilot | `.github/copilot/` |
| OpenCode | `.opencode/` o `.agent/` |

> **Tip:** La mayoría auto-detectan `.agent/skills/`

## 📋 Workflows Disponibles

| Comando | Descripción |
|---------|-------------|
| `/tdd` | Desarrollo guiado por tests |
| `/plan` | Planificar features complejas |
| `/code-review` | Revisar código |
| `/build-fix` | Arreglar errores de build |
| `/refactor` | Refactorizar código |
| `/generate-task` | Generar tarea desde descripción |
| `/add-knowledge` | Añadir conocimiento al sistema |

## 🤖 Agentes Especializados

| Agente | Uso |
|--------|-----|
| **Planner** | Planificación de features |
| **Architect** | Decisiones de arquitectura |
| **Java Specialist** | Desarrollo Java/Spring |
| **TDD Guide** | Tests primero |
| **Code Reviewer** | Revisión de calidad |
| **Security Reviewer** | Análisis de seguridad |
| **Build Error Resolver** | Errores de compilación |
| **Refactor Cleaner** | Limpieza de código |
| **Task Generator** | Generar tareas |
| **React Specialist** | Frontend React |
| **Knowledge Updater** | Gestión de conocimiento |
| **E2E Runner** | Tests E2E con Playwright |

## ⚙️ Configuración Java

El sistema está pre-configurado para proyectos Java con:

- ✅ **Lombok** obligatorio (`@RequiredArgsConstructor`, `@Slf4j`, `@Builder`)
- ✅ **MapStruct** obligatorio para mapeos Entity ↔ DTO
- ✅ **Spring Boot 3.x** con patrones estándar
- ✅ **TDD** con cobertura mínima 80%
- ✅ **Redis** para caching

## 📖 Uso

El agente lee automáticamente `.agent/AGENTS.md` al iniciar. Este archivo define:

1. **Bootstrap** - Detectar OS, cargar estado
2. **Memory System** - CONTEXT, LEARNINGS, QUEUE, LOGS
3. **Task Queue** - Priorización y dependencias
4. **Recovery** - 5 niveles de auto-recuperación
5. **Communication** - Protocolo con el usuario

## 🤝 Contribuir

1. Fork el repo
2. Crea tu skill/agente en la carpeta correspondiente
3. Sigue el formato existente (frontmatter YAML + markdown)
4. Submit PR

## 📄 Licencia

MIT

## 🙏 Créditos

- [everything-claude-code](https://github.com/affaan-m/everything-claude-code)
- [antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills)
- [awesome-copilot](https://github.com/github/awesome-copilot)
