# Diseño — Plugin Claude Code: Framework de Calidad de Repositorios

**Fecha:** 2026-06-05
**Autor:** Diego Mahecha
**Estado:** Aprobado para implementación

## Propósito

Plugin de Claude Code que ejecuta análisis de calidad sobre cualquier repositorio aplicando el *Framework de Criterios — Análisis de Calidad de Repositorio v1.0* (216 criterios, 23 dimensiones + plantillas). Mide con herramientas reales cuando están disponibles y usa el juicio de Claude (leyendo el código) cuando no, generando reportes estructurados en el repo analizado.

## Decisiones clave (resueltas en brainstorming)

| Decisión | Elección |
|----------|----------|
| Función principal | Ejecutar análisis y generar reportes automáticamente |
| Modo de medición | Híbrido: herramientas disponibles + juicio de Claude para el resto |
| Alcance | Modos diferenciados (no siempre los 216) |
| Modos | `quick`, `full`, `security`, `deep-dive` |
| Destino de reportes | `./quality-report/YYYY-MM-DD/` en el repo analizado |
| Empaquetado de criterios | Bundle: docs del framework copiados dentro del plugin (autónomo) |
| Arquitectura | Commands delgados + 1 skill compartida `quality-framework` |

## Arquitectura

Enfoque: **Commands + Skill compartida**. Los 4 slash commands son delgados; delegan en la skill `quality-framework` pasando el modo. La skill concentra todo el conocimiento (criterios, scoring, plantillas, lógica híbrida). Sin duplicación entre commands.

### Estructura de archivos

```
quality-framework-plugin/
├── .claude-plugin/
│   └── plugin.json              # manifest
├── commands/
│   ├── quality-quick.md         # modo por-PR (subset)
│   ├── quality-full.md          # 216 criterios + §20 + §22
│   ├── quality-security.md      # §06 + §13
│   └── quality-deep-dive.md     # §25 + §26 sobre un hallazgo
└── skills/
    └── quality-framework/
        ├── SKILL.md             # lógica central: flujo, scoring, output, híbrido
        └── references/
            ├── 01-estructura-gobernanza.md … 24-sdd-madurez.md   # 24 docs de criterios
            ├── scoring.md       # umbrales + pesos §22 (verde/ámbar/rojo, 0-10)
            ├── herramientas.md  # §19 matriz de herramientas (.NET/TS/universal)
            ├── antipatrones.md  # §21 catálogo
            └── plantillas.md    # §20 informe, §25 profundidad, §26 arreglos
```

Los `references/` se copian desde los .md existentes del framework (bundle autónomo: funciona en cualquier máquina sin el repo framework presente).

### Manifest `plugin.json`

```json
{
  "name": "framework-calidad",
  "version": "1.0.0",
  "description": "Análisis de calidad de repositorios — 216 criterios, 23 dimensiones",
  "author": { "name": "Diego Mahecha" }
}
```

## Flujo de ejecución (híbrido)

Cada command corre este flujo vía la skill:

1. **Detectar repo objetivo** — arg `path` del command o `cwd`. Detectar stack (.NET/TS/otro) leyendo manifests (`*.csproj`, `package.json`).
2. **Inventario de herramientas** — chequear qué hay disponible (gitleaks, sonar, coverlet, eslint, k6…) según §19. Loguear cuáles correrán vs cuáles faltan.
3. **Medir** — correr herramientas disponibles vía Bash, capturar salida.
4. **Evaluar criterios** — por cada criterio del modo: si hubo herramienta → dato real contra umbral; si no → juicio de Claude leyendo código. Marcar fuente.
5. **Scorear** — aplicar pesos §22 según tipo de proyecto (crítico/MVP/legacy). Verde/ámbar/rojo por criterio, score global 0-10.
6. **Escribir reporte** — `./quality-report/YYYY-MM-DD/` en el repo analizado, plantilla §20 (+ §25/§26 en deep-dive).
7. **Resumen en chat** — score global + top hallazgos + ruta del archivo.

**Transparencia:** cada hallazgo marca su fuente — 🔧 medido (herramienta) vs 🧠 estimado (juicio de Claude).

## Modos y salidas

| Modo | Command | Criterios | Salida en `./quality-report/YYYY-MM-DD/` |
|------|---------|-----------|------------------------------------------|
| **quick** | `/quality-quick [path]` | Subset por-PR de alto impacto: §05, §06 (secrets), §07 (cobertura), §10 | `quick-report.md` (resumen + flags) |
| **full** | `/quality-full [path] [tipo]` | 216 completos, 23 dimensiones | `informe-completo.md` (§20) + `score.md` (§22) |
| **security** | `/quality-security [path]` | §06 + §13 | `security-report.md` |
| **deep-dive** | `/quality-deep-dive [path] "hallazgo"` | §25 análisis + §26 arreglos sobre 1 hallazgo | `deep-dive-<slug>.md` + `arreglos.md` |

### Argumentos

- `path` — opcional, default `cwd`.
- `tipo` (solo `full`) — `critico` / `mvp` / `legacy`: elige la matriz de pesos §22. Default `mvp`.
- `"hallazgo"` (solo `deep-dive`) — descripción del problema a investigar.

## Manejo de errores y casos borde

- **No es repo git / path inválido** → abortar con mensaje claro.
- **Sin herramientas instaladas** → seguir, todo 🧠 estimado, avisar al inicio.
- **Herramienta falla o timeout** → marcar el criterio 🧠 estimado, no abortar.
- **Repo enorme** → `full` despacha dimensiones a subagents en paralelo (evita saturar contexto).
- **Stack no reconocido** → aplicar solo criterios universales, avisar.
- **Carpeta de reporte ya existe** → sufijo `-2`, nunca sobrescribir.

## Criterios de éxito

- `/quality-full` sobre el monorepo de ejemplo produce `informe-completo.md` + `score.md` consistentes con el patrón §23 (GOP 360°).
- Cada hallazgo identifica su fuente (medido vs estimado).
- El plugin funciona sin el repo framework presente (autónomo vía bundle).
- Modos `quick`/`security`/`deep-dive` producen sus salidas correspondientes.
- Sin herramientas instaladas, el plugin degrada a 100% estimado sin fallar.

## Fuera de alcance (YAGNI)

- Instalar herramientas faltantes automáticamente.
- Modo `score`-only o selección por dimensión arbitraria.
- Integración CI/CD (el plugin se ejecuta interactivo desde Claude Code).
- Histórico/diff entre reportes.
