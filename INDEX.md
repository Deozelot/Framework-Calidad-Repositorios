# Framework de Criterios — Análisis de Calidad de Repositorio
**Versión:** 1.0
**Fecha:** 2026-06-05
**Aplicable a:** Monorepos multi-stack con SDLC formal (origen: GOP 360°)

---

## Propósito

Catálogo exhaustivo de **23 dimensiones** y **200+ criterios accionables** para producir informes fieles del estado de un repositorio. Cada criterio es medible, con herramienta sugerida y umbral verde/rojo.

---

## Cómo Navegar

| Audiencia | Categorías recomendadas |
|---|---|
| **Ejecutivo** | [§00](00-como-usar.md) + [§22 Score](22-score-global.md) + [§23 Aplicación GOP](23-aplicacion-gop360.md) |
| **Arquitecto** | §02 + §03 + §04 + §16 + §17 + §24 |
| **Tech Lead** | §03 + §05 + §07 + §08 + §17 + §26 |
| **DevSecOps** | §06 + §08 + §11 + §13 + §14 |
| **QA Lead** | §07 + §10 + §17 |
| **Auditoría regulatoria** | §06 + §08 + §09 + §10 + §14 |
| **Onboarding nuevo dev** | §00 + §01 + §02 + §09 + §10 |

---

## Tabla de Contenidos

### Guía de uso
- [§00 — Cómo usar el framework](00-como-usar.md)

### Estructura y Arquitectura
- [§01 — Estructura y Gobernanza del Repo](01-estructura-gobernanza.md)
- [§02 — Arquitectura](02-arquitectura.md)
- [§03 — Mantenibilidad: Tamaño y Cohesión](03-mantenibilidad.md)
- [§04 — Cohesión y Acoplamiento](04-cohesion-acoplamiento.md)

### Código
- [§05 — Buenas Prácticas de Código](05-buenas-practicas-codigo.md)

### Seguridad y Calidad
- [§06 — Seguridad](06-seguridad.md)
- [§07 — Testing y Cobertura](07-testing-cobertura.md)

### Operación
- [§08 — CI/CD y Automatización](08-cicd.md)
- [§11 — Observabilidad y Operación](11-observabilidad.md)
- [§12 — Performance y Escalabilidad](12-performance.md)

### Documentación y Proceso
- [§09 — Documentación](09-documentacion.md)
- [§10 — Procesos y Trazabilidad](10-procesos-trazabilidad.md)

### Dependencias y Datos
- [§13 — Dependencias y Supply Chain](13-dependencias.md)
- [§14 — Calidad de Datos y Persistencia](14-datos-persistencia.md)

### Stack-específico
- [§15 — Frontend (Angular/React/Vue)](15-frontend.md)
- [§16 — Backend (.NET/Node/Java)](16-backend.md)

### Riesgo y Métricas
- [§17 — Riesgos y Deuda Técnica](17-deuda-tecnica.md)
- [§18 — Métricas Cuantitativas Recomendadas](18-metricas-cuantitativas.md)

### Toolkit
- [§19 — Herramientas Recomendadas por Categoría](19-herramientas.md)
- [§20 — Plantilla de Informe](20-plantilla-informe.md)
- [§21 — Anti-patrones: Señales de Alerta](21-antipatrones.md)
- [§22 — Cálculo del Score Global](22-score-global.md)

### Caso de aplicación
- [§23 — Aplicación a GOP 360°](23-aplicacion-gop360.md)

### Proceso SDD y Remediación
- [§24 — Madurez Spec-Driven Development](24-sdd-madurez.md)
- [§25 — Plantilla: Análisis a Profundidad](25-analisis-profundidad.md)
- [§26 — Plantilla: Arreglos Priorizados](26-arreglos-priorizados.md)

---

## Resumen Cuantitativo

| § | Categoría | Criterios |
|---|---|---|
| 01 | Estructura/Gobernanza | 10 |
| 02 | Arquitectura | 10 |
| 03 | Mantenibilidad | 10 |
| 04 | Cohesión/Acoplamiento | 7 |
| 05 | Buenas prácticas código | 15 |
| 06 | Seguridad | 17 |
| 07 | Testing/Cobertura | 14 |
| 08 | CI/CD | 15 |
| 09 | Documentación | 14 |
| 10 | Procesos/Trazabilidad | 15 |
| 11 | Observabilidad | 10 |
| 12 | Performance | 11 |
| 13 | Dependencias | 10 |
| 14 | Datos/Persistencia | 10 |
| 15 | Frontend | 12 |
| 16 | Backend | 11 |
| 17 | Deuda/Riesgos | 10 |
| 24 | SDD (madurez) | 15 |
| **Total criterios** | | **216** |

> §25 y §26 son **plantillas de entregable** (no dimensiones con criterios) — no suman al conteo.

---

## Convenciones del Framework

- **Verde** — Cumple. Mantener.
- **Ámbar** — Aceptable. Plan de mejora documentado.
- **Rojo** — Crítico. Acción inmediata.
- **MUST/SHOULD/MAY** — RFC 2119 cuando aplica.
- **Score 0-10** — 9-10 élite / 7-8 sano / 5-6 aceptable / 3-4 riesgoso / 0-2 crítico.

---

## Historial de Versiones

| Versión | Fecha | Cambios |
|---|---|---|
| 1.0 | 2026-06-05 | Versión inicial — 23 secciones, 201 criterios |
| 1.1 | 2026-06-05 | +§24 SDD (15 criterios), +§25 Análisis Profundidad, +§26 Arreglos Priorizados (secciones agnósticas, reutilizables entre proyectos) — total 216 criterios |
