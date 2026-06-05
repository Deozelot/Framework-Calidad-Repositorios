# §24 — Madurez Spec-Driven Development (SDD)

[← Volver al índice](INDEX.md) · Anterior: [§23](23-aplicacion-gop360.md) · Siguiente: [§25](25-analisis-profundidad.md)

---

## Objetivo

Evaluar la madurez del **proceso de desarrollo dirigido por especificaciones** — qué tan sistemáticamente el proyecto deriva su código de specs, cómo gobierna ese proceso, y cuánto diverge la implementación de lo especificado.

A diferencia de §01-§17 (que miden calidad de **producto**), §24 mide calidad de **proceso**. Por eso su resultado es un **Índice de Madurez** informativo, separado del score global (ver [§22](22-score-global.md)).

---

## Aplicabilidad

Aplica a proyectos con un proceso formal de especificación, en cualquiera de sus formas:

- Herramienta de specs (Spec Kit, o equivalente que genere specs/plans/tasks)
- RFCs / Design Docs versionados
- ADRs (Architecture Decision Records)
- Agentes / orquestación de desarrollo
- Constitución / documento de governance técnica vinculante

**Si el proyecto NO usa ningún proceso spec-driven → marcar §24 como N/A.** No penalizar: un proyecto puede ser sano (§01-§17 altos) sin proceso formal de specs. §24 solo aplica donde el proceso existe y se quiere medir su salud.

---

## Las 5 áreas

Cada área se puntúa 0-10 de forma independiente.

### 24.1 — Setup de herramienta de specs

¿Existe y está completa la infraestructura para producir specs?

| Qué medir | Cómo medir |
|---|---|
| Templates de spec/plan/tareas completos | Inventario de plantillas; LOC por template (no esqueletos vacíos) |
| Automatización de scaffolding | Scripts/comandos que crean estructura de feature |
| Workflow definido con gates | Existe un flujo `spec → diseño → tareas → implementación` con puntos de revisión |
| Integración con la herramienta de trabajo | Config presente y funcional |

*Ejemplo de herramienta: Spec Kit. Alternativa válida: plantillas RFC + checklist de PR. La herramienta concreta no importa — importa que el setup exista y sea usable.*

### 24.2 — Adherencia al flujo spec→código

¿Los cambios reales siguen el ciclo definido?

| Qué medir | Cómo medir |
|---|---|
| Cobertura de artefactos por feature | % de features con spec + diseño + tareas presentes |
| Profundidad de specs | LOC/detalle por spec — specs reales vs esqueletos |
| Gates respetados | Evidencia de que los puntos de revisión se ejecutan |

### 24.3 — Automatización / agentes *(opcional)*

Solo si el proyecto usa orquestación automatizada (agentes, validación de specs en CI, generación asistida).

| Qué medir | Cómo medir |
|---|---|
| Roles/capacidades definidos | Estructura de agentes o pipeline documentada |
| Routing / delegación | Reglas de qué proceso maneja qué tipo de solicitud |
| Capacidades documentadas | Skills/funciones con criterio de aceptación |

**Si el proyecto no usa automatización → excluir 24.3 del promedio.** No penalizar; el desarrollo manual con buen proceso de specs es válido.

### 24.4 — Governance / constitución

¿Hay reglas vinculantes que gobiernan el desarrollo?

| Qué medir | Cómo medir |
|---|---|
| Completitud del documento de governance | Cobertura de negocio + arquitectura + reglas no negociables |
| Principios claros | Reglas explícitas (MUST/SHOULD/MAY) |
| Trazabilidad activa | % de unidades de trabajo con trazabilidad registrada |
| Versionado coherente | Una fuente de verdad; versiones sincronizadas si hay copias |

### 24.5 — Divergencia spec↔código (drift)

¿El código refleja lo especificado?

| Qué medir | Cómo medir |
|---|---|
| Alineación de contratos | Muestreo: rutas/APIs/entidades declaradas en spec vs implementadas |
| Drift de convenciones | Inconsistencias entre features de distinta época |
| Specs huérfanas / código sin spec | Specs que ya no reflejan el código; código sin spec asociada |
| Gates de QA persistidos | Los checklists/aprobaciones dejan artefacto auditable |

---

## Cómo medir (comandos genéricos)

```bash
# 24.1 — inventario de tooling
find <specs-tooling-dir> -type f
wc -l <templates>/*

# 24.2 — cobertura de artefactos por feature
for f in <specs-dir>/*/; do
  # verificar presencia de spec/diseño/tareas
done

# 24.4 — governance
wc -l <governance-docs>
# verificar versionado y única fuente de verdad

# 24.5 — drift (muestreo 2-3 features)
# comparar contratos/rutas declaradas vs implementadas
grep -rn "<convención declarada>" <code>   # ¿se usa consistentemente?
```

---

## Umbrales por área

| Sub-score | Significado |
|---|---|
| 9-10 | Optimizado — automatizado, sin drift, gates persistidos |
| 7-8 | Definido y consistente — drift menor, proceso sistemático |
| 5-6 | Aplicado con gaps — proceso existe pero inconsistente |
| 3-4 | Parcial — no sistemático |
| 0-2 | Ausente o roto |

---

## Índice de Madurez SDD

```
Índice de Madurez SDD = promedio de las áreas APLICABLES
```

- Excluir 24.3 del promedio si el proyecto no usa automatización.
- El índice es **INFORMATIVO**. Mide proceso, no producto. **NO entra al score global de calidad** (ver [§22](22-score-global.md) — fila informativa).
- Reportar como número separado: "Índice de Madurez SDD: X.X/10".

### Escala de interpretación

| Rango | Etiqueta |
|---|---|
| 9-10 | Optimizado |
| 7-8 | Definido y consistente |
| 5-6 | Aplicado con gaps |
| 3-4 | Parcial |
| 0-2 | Ausente |

---

## Identificadores de hallazgo

`C-24-XX` (crítico) · `M-24-XX` (medio) · `B-24-XX` (bajo) — consistente con [§20](20-plantilla-informe.md).

---

## Anti-patrones SDD

- **Spec-as-documentation:** specs escritas *después* del código — documentación, no spec-driven
- **Gate fantasma:** puntos de revisión que se ejecutan pero no dejan artefacto auditable
- **Múltiples fuentes de verdad:** dos documentos de governance con versionado divergente
- **Drift generacional:** convenciones distintas entre features de distinta época (ej. versionado de rutas inconsistente)
- **Governance no verificada:** reglas documentadas que ningún check de CI valida
- **Agentes/roles sin criterio de aceptación:** orquestación sin Definition of Done

---

## Referencias cruzadas

- Plantilla de informe → [§20](20-plantilla-informe.md)
- Por qué es informativa (no entra al score) → [§22](22-score-global.md)
- Derivar backlog de investigación → [§25](25-analisis-profundidad.md)
- Derivar roadmap de arreglos → [§26](26-arreglos-priorizados.md)
