# §00 — Cómo Usar el Framework

[← Volver al índice](INDEX.md)

---

## Propósito

Catalogar criterios medibles para evaluar calidad de un repositorio. Producir informes consistentes y comparables en el tiempo.

---

## Estructura de cada criterio

| Campo | Descripción |
|---|---|
| **Qué medir** | Variable concreta observable |
| **Cómo medir** | Herramienta / comando / inspección |
| **Verde / Rojo** | Umbral aceptable y crítico |
| **Evidencia** | Qué adjuntar al informe |

---

## Flujo de uso recomendado

1. **Seleccionar audiencia** — define qué secciones cubrir
2. **Ejecutar comandos** — listados en cada §
3. **Aplicar umbrales** — comparar valor real vs verde/rojo
4. **Documentar evidencia** — comandos ejecutados + commit SHA analizado
5. **Calcular score** — usar [§22](22-score-global.md)
6. **Redactar informe** — plantilla [§20](20-plantilla-informe.md)
7. **Priorizar acción** — P1/P2/P3 según severidad

---

## Frecuencia de aplicación

| Cadencia | Cobertura |
|---|---|
| **Por PR** | §05 buenas prácticas + §07 cobertura del cambio |
| **Semanal** | §06 secrets/vulnerabilidades + §13 deps |
| **Sprint** | §03 LOC + §07 cobertura global + §08 CI metrics |
| **Trimestral** | Framework completo — score global + §24 madurez SDD |
| **Pre-release** | §06 seguridad + §07 E2E + §12 performance |
| **Por ronda de análisis** | §25 backlog de profundidad + §26 arreglos priorizados (derivados del informe) |

---

## Audiencias y secciones priorizadas

| Audiencia | Categorías obligatorias |
|---|---|
| Ejecutivo / Cliente | §00 + §22 + §23 |
| Arquitecto | §02, §03, §04, §16, §17, §24 |
| Tech Lead | §03, §05, §07, §08, §17, §26 |
| DevSecOps | §06, §08, §11, §13, §14 |
| QA Lead | §07, §10, §17 |
| Auditoría regulatoria | §06, §08, §09, §10, §14, §25 |
| Onboarding nuevo dev | §00, §01, §02, §09, §10 |

---

## Reglas de oro

1. **Medir antes de opinar** — comando ejecutado > intuición
2. **Adjuntar evidencia** — informe sin evidencia = opinión
3. **Comparar contra umbral** — número absoluto sin referencia es ruido
4. **Documentar commit/branch** — métricas evolucionan
5. **Priorizar** — no todo es crítico; rojo > ámbar > verde
6. **Iterar** — primer informe sirve de baseline, siguientes miden delta
