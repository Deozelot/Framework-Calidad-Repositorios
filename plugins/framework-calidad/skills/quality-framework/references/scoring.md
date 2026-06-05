# §22 — Cálculo del Score Global

[← Volver al índice](INDEX.md) · Anterior: [§21](21-antipatrones.md) · Siguiente: [§23](23-aplicacion-gop360.md)

---

## Objetivo

Convertir métricas dispersas en un score único 0-10 comparable entre informes.

---

## Fórmula base

```
score_global = Σ (score_dimensión × peso_dimensión)
```

Donde:
- `score_dimensión` ∈ [0, 10]
- `peso_dimensión` ∈ [0, 1] con Σ pesos = 1

---

## Pesos sugeridos por tipo de proyecto

| Dimensión | Producto crítico (regulado) | MVP / Startup | Legacy / Maintenance |
|---|---|---|---|
| 01 Estructura | 5% | 5% | 10% |
| 02 Arquitectura | 15% | 10% | 20% |
| 03 Mantenibilidad | 15% | 10% | 25% |
| 04 Cohesión/Acoplamiento | 5% | 5% | 5% |
| 05 Buenas prácticas código | 5% | 5% | 5% |
| 06 Seguridad | 20% | 15% | 10% |
| 07 Testing/Cobertura | 15% | 5% | 15% |
| 08 CI/CD | 5% | 15% | 5% |
| 09 Documentación | 5% | 5% | 10% |
| 10 Procesos | 5% | 5% | 5% |
| 11 Observabilidad | 5% | 5% | 5% |
| 12 Performance | — | 5% | — |
| 13 Dependencias | — | 5% | — |
| 14 Datos | — | 5% | — |
| **Total** | **100%** | **100%** | **100%** |
| 24 SDD (madurez) | informativo | informativo | informativo |

> **§24 SDD es informativa.** Mide madurez de **proceso** de desarrollo, NO calidad de **producto**. Se reporta como "Índice de Madurez SDD" separado y **NO entra al ponderado**. La fórmula y los pesos de §01-§14 (suma 100%) no cambian al incluirla.

### Justificación de pesos

- **Producto crítico (ANH, salud, financiero):** prioriza Seguridad + Arquitectura + Testing porque fallar tiene impacto regulatorio/legal
- **MVP/Startup:** prioriza CI/CD + Performance porque velocidad y producto-market-fit son críticos
- **Legacy/Maintenance:** prioriza Mantenibilidad + Documentación + Arquitectura porque el costo de cambio es alto

---

## Cómo calcular score de cada dimensión

### Método A — Promedio de criterios (cuantitativo puro)

Para cada criterio dentro de la dimensión:
- Cumple verde → 10
- Cumple ámbar → 5
- Cumple rojo → 0

```
score_dim = (Σ scores_criterios) / N_criterios
```

### Método B — Ponderado por importancia

Algunos criterios pesan más:

```
score_dim = Σ (score_criterio × peso_criterio_interno)
```

### Método C — Híbrido (recomendado)

- Criterios "must" (críticos): si alguno está rojo, score_dim máximo = 5
- Resto: promedio normal

Ejemplo §06 Seguridad:
- "Secrets en código" rojo → score_dim ≤ 5 sin importar otros criterios
- "Hashing MD5 para passwords" rojo → score_dim ≤ 3

---

## Escala de interpretación

| Rango | Etiqueta | Significado | Color |
|---|---|---|---|
| 9.0–10.0 | **Élite** | Referencia industria. Mantener. | 🟢 |
| 7.0–8.9 | **Sano** | Producción confiable. Mejoras menores. | 🟢 |
| 5.0–6.9 | **Aceptable** | Funcional. Refactor planeado. | 🟡 |
| 3.0–4.9 | **Riesgoso** | Refactor urgente. Riesgo activo. | 🟠 |
| 0.0–2.9 | **Crítico** | Rescate estructural. Considerar rewrite. | 🔴 |

---

## Ejemplo de cálculo completo

Proyecto regulado con pesos columna A:

| § | Categoría | Score dim | Peso | Aporte |
|---|---|---|---|---|
| 01 | Estructura | 6.5 | 5% | 0.325 |
| 02 | Arquitectura | 9.0 | 15% | 1.350 |
| 03 | Mantenibilidad | 6.0 | 15% | 0.900 |
| 04 | Cohesión | 7.5 | 5% | 0.375 |
| 05 | Buenas prácticas | 7.0 | 5% | 0.350 |
| 06 | Seguridad | 5.5 | 20% | 1.100 |
| 07 | Testing | 5.0 | 15% | 0.750 |
| 08 | CI/CD | 6.0 | 5% | 0.300 |
| 09 | Documentación | 7.0 | 5% | 0.350 |
| 10 | Procesos | 9.0 | 5% | 0.450 |
| 11 | Observabilidad | 4.0 | 5% | 0.200 |
| **Total** | | | **100%** | **6.45/10** |

**Veredicto:** Aceptable, refactor planeado. Foco en §07 Testing + §06 Seguridad + §11 Observabilidad.

---

## Reglas de override (penalizaciones automáticas)

Independientemente del cálculo ponderado, **el score global se capa** si:

| Condición | Score global máximo |
|---|---|
| Secrets activos en repo (no rotados) | 3.0 |
| Vulnerabilidad CRITICAL sin patch >30 días | 4.0 |
| Sin backups o sin test de restore | 5.0 |
| Sin CI activo / build rojo persistente | 5.0 |
| Sin Auth en endpoints sensibles | 4.0 |
| Force push permitido en main | 5.0 |
| Cobertura <30% en módulo crítico | 6.0 |
| 1 contributor único (bus factor 1) en módulo crítico | 7.0 |

Razón: la fórmula ponderada puede dar 7/10 mientras un riesgo crítico ignorado vacía la confianza. El cap refleja esa realidad.

---

## Tendencia entre informes

Más importante que el número absoluto es la **tendencia**:

| Patrón | Interpretación |
|---|---|
| ↑↑↑ (3 informes consecutivos al alza) | Equipo invierte en calidad, sano |
| → (estable) | Mantiene baseline, OK si baseline alto |
| ↓ (decremento) | Alarma — investigar causa raíz |
| ↑↓↑↓ (oscilante) | Sin estrategia coherente |

---

## Visualización recomendada

### Spider/Radar chart por dimensión

Muestra fortalezas/debilidades de un vistazo:

```
              Estructura
                 9
       Proc 9 ╱─────╲ 8 Arq
            │       │
   Docs 7 ◀─┤   X   ├─▶ 6 Mant
            │       │
        CI 6 ╲─────╱ 6 Sec
                 5
              Tests
```

### Time series del score global

Eje X: fecha informe / Eje Y: score 0-10

Identifica:
- Regresiones tras releases
- Mejora tras refactor sprints
- Estancamiento

### Heatmap de dimensiones

Filas = informes, Columnas = §s, Color = score
- Rojo persistente en columna §X = problema crónico
- Verde universal = repo elite

---

## Score por módulo (para monorepos)

Cuando el repo tiene N módulos:

```
score_global = Σ (score_módulo × LOC_módulo) / Σ LOC_módulo
```

Promedio ponderado por tamaño — módulos grandes pesan más en el promedio.

Ejemplo:

| Módulo | Score | LOC | Aporte |
|---|---|---|---|
| Auth | 7.6 | 5,800 | 7.6 × 5,800 = 44,080 |
| Operations | 5.0 | 14,669 | 5.0 × 14,669 = 73,345 |
| Produccion | 6.6 | 6,280 | 6.6 × 6,280 = 41,448 |
| **Total** | | **26,749** | **158,873** |

`score_global = 158,873 / 26,749 = 5.94`

Operations arrastra el promedio por tamaño.

---

## Auditoría del propio scoring

Cada 6 meses revisar:

- ¿Los pesos siguen reflejando prioridades del negocio?
- ¿Algún criterio se volvió irrelevante / obsoleto?
- ¿Aparecieron nuevas dimensiones (e.g., AI ethics, sustainability)?
- ¿Los umbrales siguen alineados con industria?

El framework es vivo. Versionar cambios.

---

## Referencias cruzadas

- Cómo medir cada §N → secciones individuales del framework
- Plantilla con scores → [§20 Plantilla informe](20-plantilla-informe.md)
- Aplicación práctica → [§23 GOP 360°](23-aplicacion-gop360.md)
