# §20 — Plantilla de Informe

[← Volver al índice](INDEX.md) · Anterior: [§19](19-herramientas.md) · Siguiente: [§21](21-antipatrones.md)

---

## Objetivo

Estructura uniforme para todo informe de calidad. Garantiza comparabilidad entre informes (mismo repo en distintos momentos, repos diferentes).

---

## Plantilla copy-paste

```markdown
# Informe de Calidad — <Nombre Repo> — <YYYY-MM-DD>

**Versión:** 1.0
**Auditor:** <Nombre>
**Commit analizado:** `<sha>` (branch `<name>`)
**Tools usadas:** <lista + versión>
**Audiencia objetivo:** <ejecutivo / técnico / regulador>

---

## Resumen Ejecutivo

- **Score global:** X.X/10
- **Top 3 riesgos críticos:**
  1. …
  2. …
  3. …
- **Top 3 fortalezas:**
  1. …
  2. …
  3. …
- **Inversión estimada para llegar a target Y.Y:** N sprints / N personas

---

## Métricas Clave

| Dimensión | Valor actual | Target | Delta vs anterior |
|---|---|---|---|
| LOC totales | | | |
| Test count | | | |
| Cobertura % | | | |
| Vulnerabilidades HIGH+ | | | |
| Build time (min) | | | |
| PRs mergeados último mes | | | |
| Score global | X.X/10 | 8.5 | — |

---

## Score por Dimensión

| § | Categoría | Score | Peso | Aporte |
|---|---|---|---|---|
| 01 | Estructura | X/10 | 5% | X.XX |
| 02 | Arquitectura | X/10 | 15% | X.XX |
| 03 | Mantenibilidad | X/10 | 15% | X.XX |
| 06 | Seguridad | X/10 | 20% | X.XX |
| 07 | Testing | X/10 | 15% | X.XX |
| 08 | CI/CD | X/10 | 10% | X.XX |
| 09 | Documentación | X/10 | 10% | X.XX |
| 10 | Procesos | X/10 | 5% | X.XX |
| 11 | Observabilidad | X/10 | 5% | X.XX |
| **Total ponderado** | | | **100%** | **X.X/10** |

---

## Hallazgos por Categoría

### §<N>. <Categoría>

**Score:** X/10
**Comandos ejecutados:**
```bash
<comando 1>
<comando 2>
```

**Métricas:**

| Indicador | Valor | Umbral verde | Umbral rojo |
|---|---|---|---|
| | | | |

**Hallazgos:**

| ID | Severidad | Hallazgo | Archivo / ubicación | Fix sugerido |
|---|---|---|---|---|
| C-NN-01 | Crítica | … | path:line | … |
| M-NN-02 | Media | … | path:line | … |

**Veredicto:** <párrafo de 2-3 líneas>

---

(repetir bloque por cada §N evaluada)

---

## Matriz de Riesgos

| Riesgo | Probabilidad | Impacto | Mitigación | Owner | Plazo |
|---|---|---|---|---|---|
| | Alta/Media/Baja | Crítico/Alto/Medio/Bajo | | | |

---

## Recomendaciones Priorizadas

### P1 — Esta semana (críticos)
- **R-01:** <título> — <descripción>
- **R-02:** …

### P2 — Próximo sprint (altos)
- **R-03:** …
- **R-04:** …

### P3 — Próximo trimestre (medios)
- **R-05:** …
- **R-06:** …

### P4 — Backlog (bajos)
- **R-07:** …

---

## Comparativa con Informe Anterior

| Métrica | Anterior (<fecha>) | Actual | Δ |
|---|---|---|---|
| Score global | | | ↑/↓/= |
| Vulnerabilidades HIGH | | | |
| Cobertura % | | | |
| LOC totales | | | |
| Bugs abiertos | | | |

**Tendencia:** ascendente / estable / descendente

---

## Apéndice A — Evidencia

### Comandos ejecutados (orden de ejecución)

```bash
# §01 — Estructura
ls -la
find . -maxdepth 2 -type d ...

# §03 — Mantenibilidad
find src -name "*.cs" -exec wc -l {} \;
...
```

### Archivos inspeccionados

| Archivo | Razón |
|---|---|
| | |

### Herramientas y versiones

| Tool | Versión | Output adjunto |
|---|---|---|
| dotnet | X.Y.Z | — |
| node | X.Y.Z | — |
| coverlet | X.Y.Z | coverage-report.html |
| gitleaks | X.Y.Z | gitleaks.json |

---

## Apéndice B — Reportes adjuntos

- `coverage-report.html`
- `gitleaks-findings.json`
- `bundle-analysis.html`
- `lighthouse-report.html`
- `dependency-tree.txt`

---

## Apéndice C — Glosario

| Término | Definición |
|---|---|
| | |

---

## Apéndice D — Próximos pasos para el auditor

1. Compartir informe con stakeholders
2. Priorizar recomendaciones con Tech Lead
3. Crear issues en tracker para P1+P2
4. Re-medir en <fecha+30d> para tracking

---

*Informe generado <YYYY-MM-DD HH:MM>. Framework v1.0.*
```

---

## Variantes recomendadas

### Variante "Ejecutiva" (1 página)

Solo:
- Resumen ejecutivo
- Score global + Top 3 riesgos
- Recomendaciones P1
- Tendencia (gráfico)

### Variante "Técnica" (profunda)

Plantilla completa + apéndices con outputs raw.

### Variante "Regulador" (compliance)

- §06 Seguridad detallada
- §10 Trazabilidad
- §14 Datos + audit log
- Evidencia notarizada (PDF firmado)

---

## Reglas de oro al redactar

1. **Cuantificar todo** — "muchos errores" es vago; "47 errors" es accionable
2. **Adjuntar evidencia** — comando ejecutado + output
3. **Priorizar** — no todo es crítico; P1>>P2>>P3
4. **Proponer fix** — cada hallazgo debe tener acción concreta
5. **Estimar costo** — "1 hora" / "1 sprint" / "1 quarter"
6. **Anclar a commit** — métricas evolucionan; siempre incluir SHA
7. **Lenguaje neutro** — no culpar personas, sí señalar prácticas
8. **Comparar vs umbral** — número solo sin referencia es ruido
9. **Visual cuando posible** — gráfico de tendencia vale 1000 palabras
10. **Audiencia primero** — ejecutivo no quiere comandos; auditor sí

---

## Anti-patrones en redacción

- "El código está mal" → "Cobertura 35% (target 80%) en módulo X"
- "Faltan tests" → "0 tests para 12 handlers críticos en Auth"
- "Refactorizar todo" → "Dividir `SaveWellHandler` 612 LOC en 3 (~200 c/u)"
- "Hay vulnerabilidades" → "3 HIGH severity en `lodash@4.17.20` → upgrade a 4.17.21"
- Lista de 50 items sin priorizar
- "Sugerimos mejorar la arquitectura" sin decir qué/cómo
- Recomendación que requiere 6 meses sin desglose en hitos

---

## Referencias cruzadas

- Cálculo score → [§22 Score global](22-score-global.md)
- Anti-patrones señalables → [§21 Antipatrones](21-antipatrones.md)
- Métricas a recoger → [§18 Métricas](18-metricas-cuantitativas.md)
- Entregables derivados del informe → [§25 Análisis Profundidad](25-analisis-profundidad.md) + [§26 Arreglos Priorizados](26-arreglos-priorizados.md)
