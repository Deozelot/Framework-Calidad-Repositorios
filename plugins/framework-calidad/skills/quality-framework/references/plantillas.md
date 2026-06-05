# Plantillas de salida

Tres plantillas de reporte. Usar según el modo (ver SKILL.md).

---

## Plantilla A — Informe completo (modo full)

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

---

## Plantilla B — Análisis en profundidad (modo deep-dive)

# §25 — Plantilla: Análisis a Profundidad

[← Volver al índice](INDEX.md) · Anterior: [§24](24-sdd-madurez.md) · Siguiente: [§26](26-arreglos-priorizados.md)

---

## Objetivo

Convertir los **pendientes de medición** y las **áreas de score bajo** de un informe de calidad en un backlog de investigación accionable. Cada línea de investigación debe terminar en una decisión, no en "investigar más".

---

## Cuándo usar

- Tras una ronda de análisis **estático** que no pudo medir todo por falta de entorno activo (sin DB, sin servidor, sin binarios)
- Cuando una dimensión salió con score bajo y se necesita **causa raíz** antes de invertir en arreglos
- Antes de presentar resultados a stakeholders, para cerrar los pendientes que afectan el score (overrides del [§22](22-score-global.md))

---

## Estructura copy-paste

```markdown
# Análisis a Profundidad — <Proyecto> — <fecha>

**Deriva de:** <informe de calidad> · **Commit:** `<sha>`

## Parte A — Pendientes de medición
## Parte B — Deep-dive áreas débiles
## Secuencia recomendada
```

### Parte A — Pendientes de medición

Por cada item, formato uniforme de 6 campos:

- **Objetivo** — qué se quiere saber
- **Herramienta** — con qué se mide
- **Comando exacto** — copy-paste ejecutable
- **Qué decisión habilita** — la decisión concreta que se desbloquea
- **Esfuerzo** — estimación
- **Origen** — §N + ID de hallazgo que lo motivó

#### Categorías universales de pendientes

Aplicables a cualquier stack — sustituir por la herramienta concreta del proyecto:

| Pendiente | Herramienta típica (genérica) | Decisión que habilita |
|---|---|---|
| Cobertura real de código | coverage tool del stack | confirma/descarta cap de score por baja cobertura |
| Load / stress test | herramienta de carga | baseline de performance bajo carga; valida cuellos de botella |
| Secret scan en historia | scanner de secrets | confirma/descarta cap por secrets en historial |
| Bundle / artifact size | analizador de bundle/artefacto | deuda de performance de entrega |
| Dependency / CVE audit | auditor de dependencias | exposición de supply chain |
| Dependencias circulares | analizador de grafo de imports | salud de la modularización |
| Accesibilidad / UX | auditor a11y/perf web | cumplimiento de estándares de UX |

### Parte B — Deep-dive áreas débiles

Por cada dimensión con score bajo, formato de 5 campos:

- **Por qué bajó** — causa observada en el informe
- **Qué investigar** — preguntas concretas a responder
- **Comando** — cómo obtener los datos
- **Qué decisión habilita** — qué se decide con el resultado
- **Origen** — §N + IDs de hallazgo

### Cierre — Secuencia recomendada

Ordenar la investigación por **impacto-en-decisión**, NO por esfuerzo:

1. Primero lo que desbloquea **overrides de score** (cap conditions del [§22](22-score-global.md)) — define si el score cambia
2. Luego lo que define **alcance de sprints** de remediación
3. Por último las verificaciones de cierre / confirmación

Presentar como tabla: `# · Investigación · Por qué primero · Qué bloquea`.

---

## Regla de oro

**Cada item DEBE nombrar la decisión concreta que habilita.** Si una investigación no desbloquea ninguna decisión, no va en el backlog — es ruido. "Investigar más X" sin decisión asociada está prohibido.

---

## Referencias cruzadas

- Cap conditions que la investigación confirma → [§22](22-score-global.md)
- Plantilla de informe base → [§20](20-plantilla-informe.md)
- Convertir hallazgos en arreglos → [§26](26-arreglos-priorizados.md)
- Madurez de proceso → [§24](24-sdd-madurez.md)

---

## Plantilla C — Arreglos priorizados (modo deep-dive)

# §26 — Plantilla: Arreglos Priorizados

[← Volver al índice](INDEX.md) · Anterior: [§25](25-analisis-profundidad.md)

---

## Objetivo

Convertir los hallazgos de un informe de calidad en un **roadmap de remediación ejecutable** — con tres niveles de detalle según la audiencia: tabla maestra (escaneo), fichas accionables (ejecución), roadmap por olas (planeación).

---

## Mapeo severidad → prioridad

Los hallazgos del informe ([§20](20-plantilla-informe.md)) ya traen severidad. Mapeo directo:

| Severidad (ID) | Prioridad | Horizonte |
|---|---|---|
| Crítica (`C-NN-XX`) | **P1** | Esta semana |
| Media (`M-NN-XX`) | **P2** | Próximo sprint |
| Baja (`B-NN-XX`) | **P3** | Próximo trimestre / backlog |

---

## Estructura — 3 niveles de detalle

### Nivel 1 — Tabla maestra

Todos los hallazgos del informe, agrupados por prioridad (P1 → P2 → P3):

```markdown
| ID | Prioridad | Hallazgo (resumen) | Esfuerzo | § origen |
|---|---|---|---|---|
| C-NN-XX | P1 | ... | 1h | §N |
```

### Nivel 2 — Fichas accionables (P1 + P2 de alto impacto)

Por cada fix prioritario, ficha de campos:

```markdown
### <ID> — <título>
- **Descripción:** qué arreglar
- **Impacto:** qué mejora / qué riesgo mitiga
- **Archivos afectados:** paths exactos
- **Pasos concretos:** secuencia numerada de acción
- **Criterio de done:** cómo verificar que quedó
- **Esfuerzo:** estimación
- **Depende de:** (si aplica) otro fix que lo bloquea
```

Los **P3 quedan solo en Nivel 1** (tabla). No requieren ficha — se documentan al implementarlos.

### Nivel 3 — Roadmap por olas

```markdown
### Ola 1 — Semana 1 (P1 + quick-wins)
| Fix | Esfuerzo | Depende de | Tipo |

### Ola 2 — Sprint 1-2 (P2 alto impacto)
| Fix | Esfuerzo | Depende de |

### Ola 3 — Trimestre (P3 + P2 restantes)
[agrupados por tema]
```

Más tres componentes de soporte:

- **Tabla de dependencias entre fixes:** `Fix · Depende de · Razón` — para no ejecutar B antes que su bloqueante A
- **Quick-wins:** lista de fixes `<2h` y alto impacto — hacer primero para momentum temprano
- **Resumen de esfuerzo por ola:** total estimado por ola

---

## Reglas de oro

1. **Acción concreta + criterio de done** — cada fix verificable, no "mejorar X"
2. **Marcar dependencias** — fix A bloquea fix B explícito
3. **Señalar quick-wins** — alto impacto / bajo esfuerzo van primero
4. **Agrupar por causa** — fixes que comparten raíz se hacen en batch
5. **Esfuerzo siempre** — "1h" / "1 sprint" / "1 trimestre"; nunca sin estimar

---

## Referencias cruzadas

- IDs y severidad de hallazgos → [§20](20-plantilla-informe.md)
- Overrides de score que los P1 pueden disparar → [§22](22-score-global.md)
- Backlog de investigación previo → [§25](25-analisis-profundidad.md)
