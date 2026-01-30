# Configuración de Agentes LLM

Este repositorio contiene la configuración para cuatro herramientas de asistencia de código:

---

## 📊 Resumen por Herramienta

| Herramienta | Archivos | Estrategia | Descripción |
|-------------|----------|------------|-------------|
| **.claude/** | 46 | Original | Configuración base con skills, agents y reglas |
| **.opencode/** | 32 | Copia 1:1 | Skills individuales idénticos a .claude |
| **.github/** | 10 | Mergeo | Agents especializados que combinan múltiples skills |
| **.agent/** | 29 | Rules + Skills | Configuración estándar de la industria (Antigravity format) |

---

## 📁 Estructura de Archivos

### .claude/ (46 archivos)
```
.claude/
├── CLAUDE.md
├── agents/
│   ├── architect.md
│   ├── debugger.md
│   └── doc-writer.md
├── rules/
│   ├── coding-style.md
│   ├── git-workflow.md
│   └── performance.md
└── skills/
    └── [26 skills con SKILL.md]
```

### .opencode/ (32 archivos)
```
.opencode/
├── AGENTS.md
├── agents/
│   ├── architect.agent.md
│   ├── debugger.agent.md
│   └── doc-writer.agent.md
└── skills/
    └── [26 skills - copia idéntica de .claude/]
```

### .github/ (10 archivos - GitHub Copilot)
```
.github/
├── copilot-instructions.md
└── agents/
    ├── architect.agent.md       ← java-design-patterns
    ├── code-review.agent.md     ← review + security-check + refactor
    ├── debugger.agent.md        ← troubleshooting + build-fix
    ├── designer.agent.md        ← theme-factory
    ├── devops.agent.md          ← docker
    ├── doc-writer.agent.md
    ├── java-backend.agent.md    ← java-coding + springboot + postgres + mongo + redis + testing + logging + api-rest + tdd
    ├── nodejs-backend.agent.md  ← nodejs-coding + postgres + mongo + redis + tdd
    └── react-frontend.agent.md  ← react-coding + frontend-patterns + react-testing + frontend-design + e2e + vercel-best-practices
```

### .agent/ (29 archivos - Google Antigravity)
```
.agent/
├── GEMINI.md                # Reglas globales
├── rules/
│   ├── coding-style.md      # Always On
│   ├── git-workflow.md      # Model Decision
│   └── performance.md       # Model Decision
├── skills/
│   ├── api-rest-design/
│   ├── build-fix/
│   ├── docker/
│   ├── e2e/
│   ├── frontend-design/
│   ├── frontend-patterns/
│   ├── java-coding/
│   ├── java-design-patterns/
│   ├── java-testing/
│   ├── logging/
│   ├── mongodb-patterns/
│   ├── nodejs-coding/
│   ├── postgres-patterns/
│   ├── prd/
│   ├── ralph/
│   ├── react-coding/
│   ├── react-testing/
│   ├── redis-patterns/
│   ├── refactor/
│   ├── review/
│   ├── security-check/
│   ├── springboot-coding/
│   ├── tdd/
│   ├── theme-factory/
│   ├── troubleshooting/
│   └── vercel-react-best-practices/
└── workflows/
    └── [Opcional: saved prompts]
```

---

## 📋 Mapeo de Contenido

### Rules (3 archivos)

| Rule | .claude/ | .opencode/ | .github/ | .agent/ |
|------|----------|------------|------------------|---------|
| coding-style | ✅ rules/ | ✅ AGENTS.md | ✅ copilot-instructions.md | ✅ rules/ |
| git-workflow | ✅ rules/ | ✅ AGENTS.md | ✅ copilot-instructions.md | ✅ rules/ |
| performance | ✅ rules/ | ✅ AGENTS.md | ✅ copilot-instructions.md | ✅ rules/ |

### Skills (26 totales)

| Skill | .claude/ | .opencode/ | .github/ | .agent/ |
|-------|----------|------------|------------------|---------|
| java-coding | ✅ | ✅ | ✅ (agents) @java-backend | ✅ |
| springboot-coding | ✅ | ✅ | ✅ (agents) @java-backend | ✅ |
| postgres-patterns | ✅ | ✅ | ✅ (agents) @java-backend, @nodejs-backend | ✅ |
| mongodb-patterns | ✅ | ✅ | ✅ (agents) @java-backend, @nodejs-backend | ✅ |
| redis-patterns | ✅ | ✅ | ✅ (agents) @java-backend, @nodejs-backend | ✅ |
| java-testing | ✅ | ✅ | ✅ (agents) @java-backend | ✅ |
| logging | ✅ | ✅ | ✅ (agents) @java-backend | ✅ |
| api-rest-design | ✅ | ✅ | ✅ (agents) @java-backend | ✅ |
| tdd | ✅ | ✅ | ✅ (agents) @java-backend, @nodejs-backend | ✅ |
| nodejs-coding | ✅ | ✅ | ✅ (agents) @nodejs-backend | ✅ |
| react-coding | ✅ | ✅ | ✅ (agents) @react-frontend | ✅ |
| react-testing | ✅ | ✅ | ✅ (agents) @react-frontend | ✅ |
| frontend-patterns | ✅ | ✅ | ✅ (agents) @react-frontend | ✅ |
| frontend-design | ✅ | ✅ | ✅ (agents) @react-frontend | ✅ |
| e2e | ✅ | ✅ | ✅ (agents) @react-frontend | ✅ |
| vercel-react-best-practices | ✅ | ✅ | ✅ (agents) @react-frontend | ✅ |
| java-design-patterns | ✅ | ✅ | ✅ (agents) @architect | ✅ |
| troubleshooting | ✅ | ✅ | ✅ (agents) @debugger | ✅ |
| build-fix | ✅ | ✅ | ✅ (agents) @debugger | ✅ |
| review | ✅ | ✅ | ✅ (agents) @code-review | ✅ |
| security-check | ✅ | ✅ | ✅ (agents) @code-review | ✅ |
| refactor | ✅ | ✅ | ✅ (agents) @code-review | ✅ |
| docker | ✅ | ✅ | ✅ (agents) @devops | ✅ |
| theme-factory | ✅ | ✅ | ✅ (agents) @designer | ✅ |
| prd | ✅ | ✅ | ❌ | ✅ |
| ralph | ✅ | ✅ | ❌ | ✅ |

### Agents (10 totales)

| Agente | .claude/ | .opencode/ | .github/ | .agent/ |
|--------|----------|------------|------------------|---------|
| architect | ✅ agents/architect.md | ✅ agents/architect.agent.md | ✅ agents/architect.agent.md | ✅ (skills) |
| debugger | ✅ agents/debugger.md | ✅ agents/debugger.agent.md | ✅ agents/debugger.agent.md | ✅ (skills) |
| doc-writer | ✅ agents/doc-writer.md | ✅ agents/doc-writer.agent.md | ✅ agents/doc-writer.agent.md | ✅ (skills) |
| java-backend | ✅ (skills) | ✅ (skills) | ✅ agents/java-backend.agent.md | ✅ (skills) |
| nodejs-backend | ✅ (skills) | ✅ (skills) | ✅ agents/nodejs-backend.agent.md | ✅ (skills) |
| react-frontend | ✅ (skills) | ✅ (skills) | ✅ agents/react-frontend.agent.md | ✅ (skills) |
| code-review | ✅ (skills) | ✅ (skills) | ✅ agents/code-review.agent.md | ✅ (skills) |
| devops | ✅ (skills) | ✅ (skills) | ✅ agents/devops.agent.md | ✅ (skills) |
| designer | ✅ (skills) | ✅ (skills) | ✅ agents/designer.agent.md | ✅ (skills) |
| **Total Agents** | **3** | **3** | **9** | **0** |

**Nota:** Antigravity no usa agents explícitos. Los skills se cargan on-demand automáticamente según el contexto.
