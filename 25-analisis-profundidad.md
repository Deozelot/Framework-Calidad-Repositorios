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
