# §17 — Riesgos y Deuda Técnica

[← Volver al índice](INDEX.md) · Anterior: [§16](16-backend.md) · Siguiente: [§18](18-metricas-cuantitativas.md)

---

## Objetivo

Cuantificar y rastrear deuda técnica acumulada, riesgos operacionales y dependencia de personas.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 17.1 | Deuda explícita registrada | `TECH_DEBT.md`, GitHub issues label |
| 17.2 | Code smells SonarQube | Count + severidad |
| 17.3 | Bugs abiertos por severidad | Issue tracker |
| 17.4 | Coverage trend (creciente/decreciente) | Time series |
| 17.5 | Test runtime trend | Time series |
| 17.6 | Build time trend | Time series |
| 17.7 | Bus factor (autores únicos commit) | `git shortlog -sn` |
| 17.8 | Hot files (más modificados) | `git log --pretty=format: --name-only \| sort \| uniq -c \| sort -rn` |
| 17.9 | Stale branches | `git for-each-ref --sort=-committerdate refs/remotes` |
| 17.10 | Orphan files (sin referencia) | Static analysis |

---

## Comandos de referencia

```bash
# Bus factor
git shortlog -sn --since="6 months ago"
git shortlog -sn --since="6 months ago" | awk 'NR==1 {first=$1} END {print first"/"NR}'

# Hot files (más modificados último año)
git log --since="1 year ago" --pretty=format: --name-only \
  | grep -v "^$" | sort | uniq -c | sort -rn | head -20

# Co-modified files (acoplamiento implícito)
git log --since="6 months ago" --name-only --pretty=format:COMMIT \
  | awk 'BEGIN{RS="COMMIT\n"} {gsub("\n", " "); print}' \
  | tr ' ' '\n' | sort | uniq -c | sort -rn | head

# Stale branches (no actualizadas en 90 días)
for branch in $(git branch -r | grep -v HEAD); do
  if [[ $(git log -1 --since="3 months ago" "$branch" | wc -l) -eq 0 ]]; then
    echo "STALE: $branch"
  fi
done

# Issues abiertas por label
gh issue list --label bug --state open
gh issue list --label "tech-debt" --state open

# Coverage trend (necesita histórico SonarCloud / Codecov)
# Manual: capturar % weekly y graficar

# Orphan TypeScript files
npx ts-prune

# Orphan .NET (sin referencias)
# Requiere análisis NDepend o Roslyn analyzer custom

# Files con > N revisiones (churn alto)
git log --since="6 months ago" --pretty=format: --name-only \
  | grep "\.cs$\|\.ts$" | sort | uniq -c | awk '$1>20' | sort -rn
```

---

## Evidencia esperada

- Top 20 hot files (más modificados)
- Bus factor por módulo (autor único = riesgo)
- Lista stale branches con último commit
- Tabla bugs abiertos por severidad/edad
- Reporte SonarQube tech debt en horas
- Lista orphan files candidatos a borrar

---

## Umbrales

| Métrica | Verde | Rojo |
|---|---|---|
| Bus factor | ≥3 | 1 |
| Stale branches | <5 | >20 |
| Bugs CRITICAL abiertos | 0 | ≥1 |
| Bugs HIGH abiertos | <5 | >15 |
| Code smells SonarQube por kLOC | <10 | >50 |
| Tech debt ratio | <5% | >15% |
| Coverage trend | creciente / estable | decreciente 3 sprints |
| Hot file revisions (6m) | <30 | >100 |
| Files sin commit > 2 años | señal sana de estabilidad | señal de muerto código |

---

## Anti-patrones de deuda

- Deuda invisible: no hay `TECH_DEBT.md` ni issues label
- "Lo arreglamos después" sin issue creada
- TODOs con > 6 meses sin atender
- Cobertura decreciente sprint tras sprint
- Build time creciendo sin contención
- Tests flaky tolerados ("solo correrlo de nuevo")
- Bus factor 1 en módulo crítico
- Hot file que cambia 50 veces/mes sin refactor
- Stale branches abandonadas
- Issues con > 1 año abiertas sin acción

---

## Tipos de deuda (Martin Fowler quadrant)

|  | Prudente | Imprudente |
|---|---|---|
| **Deliberada** | "Lanzamos ahora, refactor en Q4" | "No hay tiempo de hacerlo bien" |
| **Inadvertida** | "Ahora sabemos cómo debió hacerse" | "¿Qué es el patrón Repository?" |

Solo la **prudente deliberada** es manejable. El resto requiere educación + refactor planeado.

---

## Tech debt registry recomendado

`TECH_DEBT.md`:

```markdown
# Tech Debt Registry

| ID | Título | Severidad | Esfuerzo | Owner | Estado | Issue |
|---|---|---|---|---|---|---|
| TD-001 | DateTime.UtcNow directo en 38 sitios | Alta | 1 sprint | @diego | Open | #123 |
| TD-002 | Operations module mezcla 3 BC | Crítica | 1 quarter | @arq | Open | #124 |
| TD-003 | F101 mantiene snake_case en TS | Media | 1 sprint | @fe-lead | In Progress | #125 |
```

Política sugerida: **20% capacity sprint dedicado a tech debt**. Más sostenible que "big rewrite" cada 2 años.

---

## Refactor strategy

| Estrategia | Cuándo |
|---|---|
| **Boy Scout Rule** | Cada PR mejora algo pequeño cercano al cambio |
| **Strangler Fig** | Reemplazo gradual con nueva impl, deprecar viejo |
| **Branch by Abstraction** | Interfaz nueva, 2 impls coexisten, switch con flag |
| **Big bang refactor** | NUNCA en sistemas regulados — alto riesgo |
| **Rewrite** | Solo si el sistema es < 10K LOC o no genera revenue |

---

## DORA + SPACE metrics como leading indicators

| DORA | Qué dice |
|---|---|
| Deploy frequency | Capacidad de entrega |
| Lead time | Eficiencia del flujo |
| Change failure rate | Calidad |
| MTTR | Resiliencia |

| SPACE | Qué dice |
|---|---|
| Satisfaction | Bienestar del equipo |
| Performance | Outcomes (revenue, users, NPS) |
| Activity | Outputs (commits, PRs) — engaña sola |
| Communication | Colaboración (reviews, mentions) |
| Efficiency | Flow + interruption-free time |

---

## Referencias cruzadas

- Cobertura trend → [§07 Testing](07-testing-cobertura.md)
- Branches/PRs → [§10 Procesos](10-procesos-trazabilidad.md)
- Vulnerabilidades sin patch → [§06 Seguridad](06-seguridad.md)
- Dependencias outdated → [§13 Dependencias](13-dependencias.md)
