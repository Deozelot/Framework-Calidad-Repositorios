# §23 — Aplicación a GOP 360°

[← Volver al índice](INDEX.md) · Anterior: [§22](22-score-global.md)

---

## Objetivo

Roadmap práctico para aplicar el framework completo al repositorio GOP 360°. Mapea informes ya producidos contra cobertura del framework y prioriza próximas rondas.

> **Ejemplo de §24/§25/§26 aplicados:** El análisis de GOP 360° produjo `Analisis GOP/§18-sdd.md` (madurez SDD → [§24](24-sdd-madurez.md)), `ANALISIS-PROFUNDIDAD.md` ([§25](25-analisis-profundidad.md)) y `ARREGLOS-PRIORIDAD.md` ([§26](26-arreglos-priorizados.md)) como caso real de uso de estas secciones.

---

## Estado actual — Informes producidos

| Informe | Fecha | Cubre § | Hallazgos críticos |
|---|---|---|---|
| `analisis-estado-proyecto-gop360-*` | 2026-06-05 | §10 parcial | 11 HUs, 5 BUGs, 7% cobertura regulatoria |
| `analisis-sdd-buenas-practicas-gop360-*` | 2026-06-05 | §10 + §16 parcial | SDD divergencia, rutas sin v1, error handling controllers |
| `analisis-mantenibilidad-arquitectura-cobertura-gop360-*` | 2026-06-05 | §02 + §03 + §07 | 6 handlers > 200 LOC, sin TimeProvider, 17.7% handlers con test |
| `analisis-estructura-carpetas-gop360-*` | 2026-06-05 | §01 | 25 hallazgos, publish/ 56MB, HU-010 colisión |
| `analisis-modulos-backend-gop360-*` | 2026-06-05 | §02 + §16 | Operations 3 BC mezclados, 48/48 visibility split |
| `framework-criterios-analisis-calidad-repo.md` | 2026-06-05 | — (meta) | 23 categorías, 201 criterios |

---

## Cobertura del framework — mapa de gaps

| § | Categoría | Estado | Próxima acción |
|---|---|---|---|
| 01 | Estructura | ✅ Cubierto | Re-medir post-Fase 1 refactor |
| 02 | Arquitectura | ✅ Cubierto | — |
| 03 | Mantenibilidad | ✅ Cubierto | Re-medir post-refactor handlers |
| 04 | Cohesión/Acoplamiento | ⚠️ Parcial | Medir LCOM + cíclicos TS |
| 05 | Buenas prácticas | ⚠️ Parcial | grep completo `any`/console/Result |
| 06 | Seguridad | ❌ Pendiente | **PRIORIDAD ALTA** — gitleaks + dotnet --vulnerable + audit Authorize |
| 07 | Testing/Cobertura | ⚠️ Parcial (conteo) | Ejecutar `coverlet` real + reporte HTML |
| 08 | CI/CD | ❌ Pendiente | Inspección `.github/workflows/` + métricas builds |
| 09 | Documentación | ⚠️ Mencionado | Audit XML docs + Swagger reachability |
| 10 | Procesos | ✅ Cubierto | Medir lead time PR via `gh` |
| 11 | Observabilidad | ❌ Pendiente | **PRIORIDAD ALTA** — Serilog scopes + healthchecks |
| 12 | Performance | ❌ Pendiente | Bundle Angular + k6 baseline |
| 13 | Dependencias | ❌ Pendiente | `npm audit` + `dotnet list --outdated --vulnerable` |
| 14 | Datos | ⚠️ Parcial (migraciones mencionadas) | Audit FK + indexes + Global Query Filter |
| 15 | Frontend | ⚠️ Mencionado | WCAG audit + Lighthouse CI |
| 16 | Backend | ✅ Cubierto | — |
| 17 | Deuda | ⚠️ Parcial | Bus factor + hot files |

**Cobertura framework actual:** ~50% (~9 de 17 secciones aplicables)

---

## Roadmap de próximas rondas

### Ronda 2 (próxima semana) — Crítica para producción

**Objetivo:** Cerrar gaps regulatorios + bloqueadores de producción.

| § | Acción | Comando clave |
|---|---|---|
| §06 | Audit secrets repo + historia | `gitleaks detect --log-opts="--all"` |
| §06 | CVE deps backend | `dotnet list package --vulnerable --include-transitive` |
| §06 | CVE deps frontend | `npm audit --json` |
| §06 | Endpoints sin `[Authorize]` | grep en `src/**/Controllers/*.cs` |
| §06 | Audit log calls por handler | `grep _audit.LogAsync` |
| §14 | Migraciones sin `Down()` válido | grep + manual review |
| §14 | FK sin índice | review configurations |
| §14 | Global Query Filter por entidad | grep `HasQueryFilter` |

**Entregable:** `analisis-seguridad-datos-gop360-RONDA2.md`

---

### Ronda 3 (próximo sprint) — Calidad operacional

**Objetivo:** Activar visibilidad y métricas reales en CI.

| § | Acción | Comando clave |
|---|---|---|
| §07 | Coverage real con coverlet | `dotnet test --collect:"XPlat Code Coverage"` + reportgenerator |
| §07 | Coverage frontend con vitest | `npx vitest run --coverage` |
| §08 | Workflows + branch protection | Audit `.github/` + `gh api .../protection` |
| §08 | Build success rate 30d | `gh run list --branch main --json` |
| §11 | Logging structured (Serilog) | grep `_logger.Log*` + scope analysis |
| §11 | Health checks | grep `AddHealthChecks` + curl `/health` |
| §11 | OpenTelemetry | grep `AddOpenTelemetry` |
| §13 | Deps outdated/vulnerables | comando completo §13 |

**Entregable:** `analisis-cicd-observabilidad-gop360-RONDA3.md`

---

### Ronda 4 (próximas 2 semanas) — Frontend + Performance

| § | Acción |
|---|---|
| §15 | Lighthouse CI (perf/a11y/best-practices/SEO) |
| §15 | Axe-core WCAG 2.1 AA scan |
| §15 | Bundle analyzer Angular |
| §12 | k6 load test baseline endpoints críticos |
| §12 | EF query analysis (N+1, índices) |

**Entregable:** `analisis-frontend-performance-gop360-RONDA4.md`

---

### Ronda 5 (mensual) — Tracking y deuda

| § | Acción |
|---|---|
| §17 | Bus factor + hot files |
| §17 | TECH_DEBT.md registry |
| §17 | DORA metrics (manual o 4Keys) |
| §09 | Audit ADRs, OpenAPI, Storybook |
| Comparativa | Score delta vs informes anteriores |

**Entregable:** `analisis-deuda-tracking-gop360-RONDA5.md`

---

## Score actual estimado (basado en informes producidos)

Usando pesos columna A (producto crítico regulado):

| § | Score parcial | Peso | Aporte | Notas |
|---|---|---|---|---|
| 01 | 6.5 | 5% | 0.325 | publish/, HU-010 colisión |
| 02 | 8.0 | 15% | 1.200 | NetArchTest élite, Operations BC mezclados |
| 03 | 6.0 | 15% | 0.900 | 22 handlers > 100, well-old-form 1365 |
| 04 | n/a | 5% | — | Sin medir |
| 05 | 7.0 | 5% | 0.350 | 23 any, NoWarn 1591, bajo TODO count |
| 06 | n/a | 20% | — | **Sin medir — riesgo regulatorio** |
| 07 | 5.0 | 15% | 0.750 | 17.7% handlers test, sin coverage real |
| 08 | n/a | 5% | — | Workflows existen pero T011-T015 pendientes |
| 09 | 6.0 | 5% | 0.300 | Bien interna, sin XML docs |
| 10 | 9.0 | 5% | 0.450 | Trazabilidad élite |
| 11 | n/a | 5% | — | **Sin medir** |
| **Parcial** | | **65%** | **4.275** | de 6.50 posible cubierto |

**Score normalizado parcial:** `4.275 / 6.50 × 10 = 6.58/10` (basado solo en cubierto)

**Score global proyectado:** ~6.0-6.5 si secciones pendientes son promedio. Si §06 Seguridad sale rojo, override puede capar a 5.0.

---

## Recomendaciones agregadas (consolidado de informes)

### P1 — Esta semana (críticos cross-informe)

| Origen | Recomendación |
|---|---|
| Estructura | Borrar `publish/` + agregar a `.gitignore` |
| Estructura | Renombrar `HU-010-WellExplorerSearchQuery` → `HU-012-...` |
| Backend | Aplicar migraciones 012/013/014 en RDS |
| SDD | Reconectar FiscalizacionVolumetricaFeature con NgRx store |
| SDD | Estandarizar rutas a `api/v1/` (1 de 12 cumple) |
| Backend | Visibility uniforme handlers `internal sealed` (48 archivos) |
| Backend | NetArchTest reglas para módulo Produccion |

### P2 — Próximo sprint

| Origen | Recomendación |
|---|---|
| Mant | Refactor `SaveWellAntiguoCommandHandler` 612 LOC en 3 |
| Mant | Refactor `well-old-form.ts` 1365 LOC en 10 sub-organisms |
| Mant | Introducir `TimeProvider` reemplazando 38× `DateTime.UtcNow` |
| Test | Coverage CI con coverlet + report HTML |
| FE | Eliminar `pageSize: 1000` hardcoded (2 componentes) |
| FE | Refactor camelCase F101 completo, eliminar 23 `any` |
| Estruct | Tests Auth única ubicación |
| Backend | Operations restructure `Commands/Wells|WellsAntiguo|Forma101/` |

### P3 — Próximo trimestre

| Origen | Recomendación |
|---|---|
| Arq | Decidir separación módulos `Modules/Forma101/` + `Modules/WellsAntiguo/` |
| FE | Consolidar `domains/wells` → `domains/operations/wells` |
| UIKit | Reagrupar organisms/pages por dominio (codemod) |
| Docs | XML docs en controllers + eliminar `<NoWarn>1591</NoWarn>` |
| Trazabilidad | Agrupar `Trazabilidad/{HU,BUG}/` |
| QA | Completar QA formal HU-006/008/009 |

---

## Métricas baseline para tracking

Snapshot 2026-06-05 para comparar en próximos informes:

```csv
date,total_loc_be,total_loc_fe,test_count_be,test_count_fe,
handlers_total,handlers_with_tests,handlers_over_100_loc,
controllers_over_300_loc,migrations_count,
prs_merged_30d,score_global_estimado
2026-06-05,12000,40201,418,13,
96,17,22,
4,38,
69,6.5
```

---

## Pizarra de seguimiento

Para cada ronda, actualizar:

```markdown
## Ronda <N> — <Fecha>
- Score global: X.X (Δ vs ronda anterior: +Y / -Y / =)
- §s cubiertas: <lista>
- Recomendaciones cerradas: <count>
- Recomendaciones nuevas: <count>
- Top 3 acciones próximo informe:
  1. ...
  2. ...
  3. ...
```

---

## Referencias cruzadas

- Framework completo → [INDEX](INDEX.md)
- Plantilla informes → [§20](20-plantilla-informe.md)
- Cálculo score → [§22](22-score-global.md)
