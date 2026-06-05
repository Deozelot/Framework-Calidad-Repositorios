# §10 — Procesos y Trazabilidad

[← Volver al índice](INDEX.md) · Anterior: [§09](09-documentacion.md) · Siguiente: [§11](11-observabilidad.md)

---

## Objetivo

Auditar workflow de desarrollo: branching, commits, PRs, trazabilidad spec→código, métricas de entrega.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 10.1 | Convención de branching documentada | `CONTRIBUTING.md` o convención GitFlow/trunk |
| 10.2 | Convención commits (Conventional Commits) | Lint commitlint |
| 10.3 | PRs vinculados a issue/HU | Audit últimos 20 PRs |
| 10.4 | PR description template | `.github/pull_request_template.md` |
| 10.5 | Issue templates | `.github/ISSUE_TEMPLATE/` |
| 10.6 | Versionado SemVer | Tags git |
| 10.7 | Release notes | `CHANGELOG.md` + GitHub Releases |
| 10.8 | Trazabilidad spec → código → test | Convención IDs y referencias cruzadas |
| 10.9 | HITL decisiones documentadas | Estructura trazabilidad |
| 10.10 | SDLC fases visibles | spec.md / plan.md / tasks.md per feature |
| 10.11 | Definition of Done formal | `DoD.md` por rol/tipo de tarea |
| 10.12 | Definition of Ready formal | `DoR.md` |
| 10.13 | Ritmo de entrega | Commits/PRs por semana, throughput |
| 10.14 | Lead time PR → merge | GitHub insights |
| 10.15 | MTTR (Mean Time To Recover) | Incidentes + tiempo |

---

## Comandos de referencia

```bash
# Branches activas
git branch -r --sort=-committerdate | head -20

# Stale branches > 90 días
for branch in $(git branch -r | grep -v HEAD); do
  date=$(git log -1 --format="%ar" "$branch");
  echo "$date $branch";
done | sort

# Commits por autor último mes
git shortlog -sn --since="1 month ago"

# Commits sin convención (no feat:/fix:/etc.)
git log --pretty=format:"%s" --since="1 month ago" \
  | grep -vE "^(feat|fix|chore|docs|refactor|test|style|perf|ci|build|revert)(\(.+\))?:"

# PRs cerrados último mes
gh pr list --state merged --limit 50 --json number,title,mergedAt,additions,deletions

# Lead time PR (open→merge)
gh pr list --state merged --limit 100 --json createdAt,mergedAt \
  | jq '[.[] | ((.mergedAt | fromdateiso8601) - (.createdAt | fromdateiso8601))/3600] | add/length'

# Issues vinculadas a PRs
gh pr list --state merged --limit 20 --json body \
  | jq -r '.[].body' | grep -cE "(Closes|Fixes|Resolves) #"

# Tags semver
git tag --sort=-v:refname | head
```

---

## Evidencia esperada

- Convención branching documentada (link al doc)
- % commits con prefijo Conventional Commits
- Lista PRs último mes con número/título/lead time
- Lista issues abiertas por label (bug/feature/enhancement)
- Tags semver con fecha
- Tabla SDLC: por feature, qué artefactos existen (spec/plan/tasks/code/tests/trazabilidad)

---

## Umbrales

| Métrica | Verde | Ámbar | Rojo |
|---|---|---|---|
| Commits Conventional | ≥95% | 80-94% | <80% |
| PRs vinculados a issue | ≥90% | 70-89% | <70% |
| Lead time PR→merge | <2 días | 2-7 días | >7 días |
| PR size LOC | <300 | 300-800 | >800 |
| Stale branches | 0 (cleanup ≥quincenal) | 5-10 | >20 |
| Deploy frequency | diaria | semanal | <semanal |
| MTTR | <1h | 1-4h | >24h |
| Spec→código trazabilidad | 100% features | parcial | ad-hoc |

---

## Convenciones de branching

### GitFlow (recomendado proyectos regulados)
- `main` — production
- `develop` — integration
- `feature/NNN-<slug>` — features (de develop)
- `hotfix/<slug>` — fixes urgentes (de main)
- `release/vX.Y.Z` — preparación release

### Trunk-Based (recomendado equipos pequeños/CI fuerte)
- `main` única branch long-lived
- Feature branches efímeras (<48h)
- Feature flags para work-in-progress

---

## Conventional Commits

```
<type>(<scope>): <subject>

[body]

[footer]
```

Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `style`, `perf`, `ci`, `build`, `revert`

Ejemplos:
- `feat(HU-009): add Fiscalización Volumétrica list endpoint`
- `fix(auth): correct token expiry check from < to <=`
- `refactor(operations): split SaveWellAntiguoCommandHandler`

---

## SDLC trazabilidad recomendada

```
PDF/Requirement → KB ingest →
  spec.md (Analista) →
    plan.md (Arquitecto) → research.md + ADRs →
      tasks.md (TLs) →
        código + tests →
          trazabilidad.md (estado por agente) →
            release notes
```

Cada paso referenciable por ID estable (`HU-NNN`, `BUG-NNN`, `SEC-NNN`).

---

## DORA metrics (DevOps Research and Assessment)

| Métrica | Elite | High | Medium | Low |
|---|---|---|---|---|
| Deployment frequency | On-demand | Daily-weekly | Weekly-monthly | <Monthly |
| Lead time for changes | <1h | <1d | <1w | >1m |
| Change failure rate | 0-15% | 16-30% | 16-30% | >30% |
| MTTR | <1h | <1d | <1w | >1m |

Tracking automatizable con [4Keys](https://github.com/dora-team/fourkeys) o servicios SaaS.

---

## Anti-patrones

- Commits "fix stuff", "WIP", "asdf"
- PRs gigantes (5000 LOC, 50 archivos)
- PRs sin descripción ni issue link
- Branch viva > 30 días (merge hell garantizado)
- Release sin changelog
- Hotfix directo a main sin proceso
- Tags no semver (`release-2024`, `prod-v2-final-FINAL`)
- DoD documentado pero no aplicado
- Trazabilidad solo cuando alguien pregunta
- Force push a main (catástrofe)

---

## Referencias cruzadas

- Branch protection → [§08 CI/CD](08-cicd.md)
- DoD per rol → [§09 Documentación](09-documentacion.md)
- Bus factor / autores → [§17 Deuda](17-deuda-tecnica.md)
