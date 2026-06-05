---
name: quality-framework
description: >
  Ejecuta el Framework de Calidad de Repositorios (216 criterios, 23 dimensiones)
  sobre un repo. Medición híbrida: corre herramientas reales si están disponibles,
  usa juicio leyendo código si no. Genera reportes en ./quality-report/YYYY-MM-DD/.
  Invocada por los comandos /quality-quick, /quality-full, /quality-security, /quality-deep-dive.
  Use when running a repository quality analysis or producing a quality report.
---

# Framework de Calidad — Skill central

Aplica los criterios del framework a un repositorio y produce reportes estructurados.
Los criterios completos están en `references/`. NO reproduzcas su contenido aquí; léelos
según el modo.

## Entrada

Los comandos invocan esta skill pasando: `modo`, `path` (default cwd), y opcionalmente
`tipo` (full) o `hallazgo` (deep-dive).

## Mapa de modos → criterios → salida

| Modo | Referencias a leer | Salida en ./quality-report/YYYY-MM-DD/ |
|------|--------------------|----------------------------------------|
| quick | 05, 06, 07, 10 (subset de alto impacto) | quick-report.md |
| full | 01–18, 23, 24, antipatrones, scoring | informe-completo.md + score.md |
| security | 06, 13 | security-report.md |
| deep-dive | 25/26 vía plantillas.md, + las refs relevantes al hallazgo | deep-dive-<slug>.md + arreglos.md |

## Flujo (7 pasos)

1. **Detectar repo objetivo.** Usa `path` o cwd. Verifica que sea repo git
   (`git -C <path> rev-parse`). Si no lo es o el path no existe → aborta con mensaje claro.
   Detecta stack leyendo manifests: `*.csproj`/`*.sln` (.NET), `package.json` (TS/JS).
   Stack no reconocido → solo criterios universales, avísalo.

2. **Inventario de herramientas.** Según `references/herramientas.md`, chequea cuáles están
   disponibles (p.ej. `gitleaks`, `dotnet sonarscanner`, `coverlet`, `eslint`, `k6`).
   Comando: probar `<tool> --version` o `which/where`. Loguea en chat qué correrá y qué falta.
   Sin ninguna herramienta → sigue, todo será 🧠 estimado; avísalo al inicio.

3. **Medir.** Corre las herramientas disponibles vía Bash, captura salida.
   Si una falla o se cuelga → marca esos criterios 🧠 estimado, NO abortes.

4. **Evaluar criterios.** Para cada criterio del modo (lee la referencia correspondiente):
   - Si hubo dato de herramienta → compáralo con el umbral del criterio.
   - Si no → juicio de Claude leyendo el código.
   Asigna verde/ámbar/rojo y marca la **fuente**: 🔧 medido o 🧠 estimado.

5. **Scorear.** Aplica `references/scoring.md`. En `full`, usa la matriz de pesos según
   `tipo` (critico/mvp/legacy; default mvp). Calcula score global 0–10.
   Otros modos: resumen de verde/ámbar/rojo sin score global ponderado.

6. **Escribir reporte.** Crea `./quality-report/YYYY-MM-DD/` en el repo analizado
   (fecha de hoy). Si ya existe, usa sufijo `-2`, `-3`… NUNCA sobrescribas.
   Usa la plantilla del modo (full → plantilla A de plantillas.md; deep-dive → B y C).
   Cada hallazgo lleva su emoji de fuente.

7. **Resumen en chat.** Score global (si aplica) + top hallazgos + ruta del archivo escrito.

## Reglas de transparencia

- Todo hallazgo declara fuente: 🔧 medido (herramienta) vs 🧠 estimado (juicio).
- Reporta qué herramientas faltaron y qué criterios quedaron estimados por ello.
- Nunca inventes métricas numéricas como si fueran medidas; un número estimado se marca 🧠.

## Repo grande (solo full)

Si el repo es grande, despacha la evaluación por dimensión a subagents en paralelo
(uno por grupo de referencias) y consolida sus hallazgos antes del paso 5.
