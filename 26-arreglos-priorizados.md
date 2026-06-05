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
