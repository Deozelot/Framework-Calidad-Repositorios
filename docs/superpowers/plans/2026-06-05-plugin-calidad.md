# Plugin Framework de Calidad — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a Claude Code plugin that runs the 216-criteria quality framework against any repository and writes structured reports, measuring with real tools when available and Claude's judgment otherwise.

**Architecture:** Thin slash commands (`/quality-quick`, `/quality-full`, `/quality-security`, `/quality-deep-dive`) delegate to one shared skill `quality-framework`. The skill holds all knowledge (criteria, scoring, templates, hybrid measurement logic). Framework docs are bundled as skill references so the plugin is self-contained. The repo root doubles as a local marketplace so the plugin can be installed.

**Tech Stack:** Markdown (commands, skill, references), JSON (plugin.json, marketplace.json). No build step. Verification via JSON parse + Claude Code `/plugin` loading.

**Repo layout produced:**
```
<repo-root>/                              # also the marketplace
├── .claude-plugin/
│   └── marketplace.json                  # lists the plugin
└── plugins/
    └── framework-calidad/
        ├── .claude-plugin/plugin.json
        ├── commands/
        │   ├── quality-quick.md
        │   ├── quality-full.md
        │   ├── quality-security.md
        │   └── quality-deep-dive.md
        └── skills/quality-framework/
            ├── SKILL.md
            └── references/
                ├── 00-como-usar.md … 18-metricas-cuantitativas.md
                ├── 23-aplicacion-gop360.md, 24-sdd-madurez.md
                ├── herramientas.md      # from 19
                ├── scoring.md           # from 22
                ├── antipatrones.md      # from 21
                └── plantillas.md        # from 20 + 25 + 26
```

---

### Task 1: Scaffold plugin directories and manifests

**Files:**
- Create: `plugins/framework-calidad/.claude-plugin/plugin.json`
- Create: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Create directory tree**

```bash
mkdir -p plugins/framework-calidad/.claude-plugin
mkdir -p plugins/framework-calidad/commands
mkdir -p plugins/framework-calidad/skills/quality-framework/references
mkdir -p .claude-plugin
```

- [ ] **Step 2: Write the plugin manifest**

Create `plugins/framework-calidad/.claude-plugin/plugin.json`:

```json
{
  "name": "framework-calidad",
  "version": "1.0.0",
  "description": "Análisis de calidad de repositorios — 216 criterios, 23 dimensiones. Modos: quick, full, security, deep-dive.",
  "author": { "name": "Diego Mahecha" }
}
```

- [ ] **Step 3: Write the marketplace manifest**

Create `.claude-plugin/marketplace.json`:

```json
{
  "name": "framework-calidad-marketplace",
  "owner": { "name": "Diego Mahecha" },
  "plugins": [
    {
      "name": "framework-calidad",
      "source": "./plugins/framework-calidad",
      "description": "Análisis de calidad de repositorios — 216 criterios, 23 dimensiones."
    }
  ]
}
```

- [ ] **Step 4: Verify both JSON files parse**

Run:
```bash
node -e "JSON.parse(require('fs').readFileSync('plugins/framework-calidad/.claude-plugin/plugin.json','utf8')); JSON.parse(require('fs').readFileSync('.claude-plugin/marketplace.json','utf8')); console.log('OK')"
```
Expected: `OK`

- [ ] **Step 5: Commit**

```bash
git add plugins/framework-calidad/.claude-plugin/plugin.json .claude-plugin/marketplace.json
git commit -m "feat(plugin): scaffold framework-calidad plugin + marketplace manifests"
```

---

### Task 2: Bundle framework criteria docs as skill references

The skill must reason from the 216 criteria without the framework repo present, so copy the source docs into `references/`. Special-purpose docs get friendly names; dimension-criteria docs keep their numbers. No duplication.

**Files:**
- Create (copies): `plugins/framework-calidad/skills/quality-framework/references/*.md`

- [ ] **Step 1: Copy the dimension-criteria + context docs verbatim**

```bash
DEST=plugins/framework-calidad/skills/quality-framework/references
cp 00-como-usar.md "$DEST/"
cp 01-estructura-gobernanza.md "$DEST/"
cp 02-arquitectura.md "$DEST/"
cp 03-mantenibilidad.md "$DEST/"
cp 04-cohesion-acoplamiento.md "$DEST/"
cp 05-buenas-practicas-codigo.md "$DEST/"
cp 06-seguridad.md "$DEST/"
cp 07-testing-cobertura.md "$DEST/"
cp 08-cicd.md "$DEST/"
cp 09-documentacion.md "$DEST/"
cp 10-procesos-trazabilidad.md "$DEST/"
cp 11-observabilidad.md "$DEST/"
cp 12-performance.md "$DEST/"
cp 13-dependencias.md "$DEST/"
cp 14-datos-persistencia.md "$DEST/"
cp 15-frontend.md "$DEST/"
cp 16-backend.md "$DEST/"
cp 17-deuda-tecnica.md "$DEST/"
cp 18-metricas-cuantitativas.md "$DEST/"
cp 23-aplicacion-gop360.md "$DEST/"
cp 24-sdd-madurez.md "$DEST/"
```

- [ ] **Step 2: Copy special-purpose docs to friendly names**

```bash
DEST=plugins/framework-calidad/skills/quality-framework/references
cp 19-herramientas.md "$DEST/herramientas.md"
cp 22-score-global.md "$DEST/scoring.md"
cp 21-antipatrones.md "$DEST/antipatrones.md"
```

- [ ] **Step 3: Build the combined templates reference**

```bash
DEST=plugins/framework-calidad/skills/quality-framework/references
{
  echo "# Plantillas de salida"
  echo
  echo "Tres plantillas de reporte. Usar según el modo (ver SKILL.md)."
  echo
  echo "---"
  echo
  echo "## Plantilla A — Informe completo (modo full)"
  echo
  cat 20-plantilla-informe.md
  echo
  echo "---"
  echo
  echo "## Plantilla B — Análisis en profundidad (modo deep-dive)"
  echo
  cat 25-analisis-profundidad.md
  echo
  echo "---"
  echo
  echo "## Plantilla C — Arreglos priorizados (modo deep-dive)"
  echo
  cat 26-arreglos-priorizados.md
} > "$DEST/plantillas.md"
```

- [ ] **Step 4: Verify all expected reference files exist**

Run:
```bash
ls plugins/framework-calidad/skills/quality-framework/references | wc -l
```
Expected: `25` (21 verbatim criteria/context docs: 00–18, 23, 24; plus the 4 friendly-named: herramientas, scoring, antipatrones, plantillas).

Then verify the friendly-named files are non-empty:
```bash
for f in herramientas scoring antipatrones plantillas; do
  test -s "plugins/framework-calidad/skills/quality-framework/references/$f.md" && echo "$f OK" || echo "$f EMPTY"
done
```
Expected: four `... OK` lines.

- [ ] **Step 5: Commit**

```bash
git add plugins/framework-calidad/skills/quality-framework/references
git commit -m "feat(plugin): bundle framework criteria docs as skill references"
```

---

### Task 3: Write the core skill SKILL.md

This is the brain: the 7-step hybrid flow, per-mode criteria selection, scoring, output, and error handling. It points to `references/` for criteria detail rather than inlining them.

**Files:**
- Create: `plugins/framework-calidad/skills/quality-framework/SKILL.md`

- [ ] **Step 1: Write SKILL.md**

Create `plugins/framework-calidad/skills/quality-framework/SKILL.md`:

```markdown
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
```

- [ ] **Step 2: Verify frontmatter parses and references resolve**

Run:
```bash
test -f plugins/framework-calidad/skills/quality-framework/SKILL.md && grep -q "^name: quality-framework" plugins/framework-calidad/skills/quality-framework/SKILL.md && echo "SKILL OK"
```
Expected: `SKILL OK`

Confirm every reference the skill names exists:
```bash
R=plugins/framework-calidad/skills/quality-framework/references
for f in herramientas scoring antipatrones plantillas 06-seguridad 13-dependencias; do
  test -f "$R/$f.md" && echo "$f present" || echo "$f MISSING"
done
```
Expected: six `... present` lines.

- [ ] **Step 3: Commit**

```bash
git add plugins/framework-calidad/skills/quality-framework/SKILL.md
git commit -m "feat(plugin): add core quality-framework skill"
```

---

### Task 4: Write the /quality-full command

**Files:**
- Create: `plugins/framework-calidad/commands/quality-full.md`

- [ ] **Step 1: Write the command**

Create `plugins/framework-calidad/commands/quality-full.md`:

```markdown
---
description: Análisis de calidad completo (216 criterios, 23 dimensiones) + score global. Args opcionales: [path] [tipo: critico|mvp|legacy]
---

Ejecuta un análisis de calidad COMPLETO del repositorio.

Argumentos recibidos: `$ARGUMENTS`
- Primer token = `path` del repo a analizar (opcional, default = directorio actual).
- Segundo token = `tipo` de proyecto para la matriz de pesos: `critico`, `mvp` o `legacy` (default `mvp`).

Usa la skill `quality-framework` en modo `full` con esos argumentos. Sigue su flujo de
7 pasos: detectar repo y stack, inventariar herramientas, medir, evaluar los criterios
de las dimensiones 01–18/23/24, scorear con la matriz de pesos del `tipo`, y escribir
`informe-completo.md` + `score.md` en `./quality-report/YYYY-MM-DD/` del repo analizado.
Termina con un resumen en chat: score global, top hallazgos y la ruta de los archivos.
```

- [ ] **Step 2: Verify it has frontmatter + names the skill and mode**

Run:
```bash
F=plugins/framework-calidad/commands/quality-full.md
grep -q "^description:" "$F" && grep -q "quality-framework" "$F" && grep -q "modo \`full\`" "$F" && echo "full OK"
```
Expected: `full OK`

- [ ] **Step 3: Commit**

```bash
git add plugins/framework-calidad/commands/quality-full.md
git commit -m "feat(plugin): add /quality-full command"
```

---

### Task 5: Write the /quality-quick command

**Files:**
- Create: `plugins/framework-calidad/commands/quality-quick.md`

- [ ] **Step 1: Write the command**

Create `plugins/framework-calidad/commands/quality-quick.md`:

```markdown
---
description: Análisis rápido por-PR (subset de alto impacto). Arg opcional: [path]
---

Ejecuta un análisis de calidad RÁPIDO, pensado para revisión por PR.

Argumentos recibidos: `$ARGUMENTS`
- Primer token = `path` del repo a analizar (opcional, default = directorio actual).

Usa la skill `quality-framework` en modo `quick` con ese argumento. Evalúa solo el subset
de alto impacto (dimensiones 05 buenas prácticas, 06 seguridad/secrets, 07 testing/cobertura,
10 procesos/trazabilidad). Escribe `quick-report.md` en `./quality-report/YYYY-MM-DD/` del
repo analizado y termina con un resumen en chat: flags encontrados y ruta del archivo.
```

- [ ] **Step 2: Verify**

Run:
```bash
F=plugins/framework-calidad/commands/quality-quick.md
grep -q "^description:" "$F" && grep -q "quality-framework" "$F" && grep -q "modo \`quick\`" "$F" && echo "quick OK"
```
Expected: `quick OK`

- [ ] **Step 3: Commit**

```bash
git add plugins/framework-calidad/commands/quality-quick.md
git commit -m "feat(plugin): add /quality-quick command"
```

---

### Task 6: Write the /quality-security command

**Files:**
- Create: `plugins/framework-calidad/commands/quality-security.md`

- [ ] **Step 1: Write the command**

Create `plugins/framework-calidad/commands/quality-security.md`:

```markdown
---
description: Análisis de seguridad enfocado (§06 seguridad + §13 dependencias). Arg opcional: [path]
---

Ejecuta un análisis de SEGURIDAD enfocado.

Argumentos recibidos: `$ARGUMENTS`
- Primer token = `path` del repo a analizar (opcional, default = directorio actual).

Usa la skill `quality-framework` en modo `security` con ese argumento. Evalúa las dimensiones
06 (seguridad: secrets, OWASP, auth, cifrado, supply chain) y 13 (dependencias y riesgo de
cadena de suministro). Corre las herramientas de seguridad disponibles (p.ej. gitleaks, audit
de dependencias) y marca cada hallazgo con su fuente. Escribe `security-report.md` en
`./quality-report/YYYY-MM-DD/` del repo analizado y resume en chat los hallazgos y su ruta.
```

- [ ] **Step 2: Verify**

Run:
```bash
F=plugins/framework-calidad/commands/quality-security.md
grep -q "^description:" "$F" && grep -q "quality-framework" "$F" && grep -q "modo \`security\`" "$F" && echo "security OK"
```
Expected: `security OK`

- [ ] **Step 3: Commit**

```bash
git add plugins/framework-calidad/commands/quality-security.md
git commit -m "feat(plugin): add /quality-security command"
```

---

### Task 7: Write the /quality-deep-dive command

**Files:**
- Create: `plugins/framework-calidad/commands/quality-deep-dive.md`

- [ ] **Step 1: Write the command**

Create `plugins/framework-calidad/commands/quality-deep-dive.md`:

```markdown
---
description: Investigación a fondo de un hallazgo (§25 análisis + §26 arreglos). Args: [path] "descripción del hallazgo"
---

Ejecuta una investigación EN PROFUNDIDAD sobre un hallazgo específico.

Argumentos recibidos: `$ARGUMENTS`
- Si el primer token es una ruta existente, trátalo como `path` del repo (default = directorio actual).
- El texto entre comillas (o el resto del argumento) = `hallazgo` a investigar. Es obligatorio;
  si falta, pídelo antes de continuar.

Usa la skill `quality-framework` en modo `deep-dive` con esos argumentos. Sigue las plantillas
B (análisis en profundidad, §25) y C (arreglos priorizados, §26) de `references/plantillas.md`:
analiza causa raíz del hallazgo leyendo el código y datos de herramientas, luego propón arreglos
priorizados P1/P2/P3. Escribe `deep-dive-<slug>.md` y `arreglos.md` en `./quality-report/YYYY-MM-DD/`
del repo analizado (`<slug>` derivado del hallazgo). Resume en chat la causa raíz, los arreglos
y la ruta de los archivos.
```

- [ ] **Step 2: Verify**

Run:
```bash
F=plugins/framework-calidad/commands/quality-deep-dive.md
grep -q "^description:" "$F" && grep -q "quality-framework" "$F" && grep -q "modo \`deep-dive\`" "$F" && echo "deep-dive OK"
```
Expected: `deep-dive OK`

- [ ] **Step 3: Commit**

```bash
git add plugins/framework-calidad/commands/quality-deep-dive.md
git commit -m "feat(plugin): add /quality-deep-dive command"
```

---

### Task 8: Install, smoke-test, and document usage

**Files:**
- Create: `plugins/framework-calidad/README.md`

- [ ] **Step 1: Write the plugin README**

Create `plugins/framework-calidad/README.md`:

```markdown
# framework-calidad — Plugin de Claude Code

Ejecuta el *Framework de Criterios — Análisis de Calidad de Repositorio v1.0*
(216 criterios, 23 dimensiones) sobre cualquier repositorio. Medición híbrida:
corre herramientas reales si están disponibles, usa el juicio de Claude leyendo
el código cuando no. Genera reportes en `./quality-report/YYYY-MM-DD/` del repo analizado.

## Instalación (marketplace local)

En Claude Code, desde la raíz de este repositorio:

```
/plugin marketplace add .
/plugin install framework-calidad@framework-calidad-marketplace
```

## Comandos

| Comando | Qué hace | Salida |
|---------|----------|--------|
| `/quality-quick [path]` | Subset por-PR (§05, §06, §07, §10) | `quick-report.md` |
| `/quality-full [path] [tipo]` | 216 criterios + score global. `tipo` = critico\|mvp\|legacy (default mvp) | `informe-completo.md` + `score.md` |
| `/quality-security [path]` | Seguridad (§06 + §13) | `security-report.md` |
| `/quality-deep-dive [path] "hallazgo"` | Causa raíz + arreglos P1/P2/P3 (§25 + §26) | `deep-dive-<slug>.md` + `arreglos.md` |

`path` es opcional (default = directorio actual). Cada hallazgo marca su fuente:
🔧 medido (herramienta) vs 🧠 estimado (juicio de Claude).
```

- [ ] **Step 2: Validate the whole plugin tree structurally**

Run:
```bash
node -e "
const fs=require('fs');
const must=[
 'plugins/framework-calidad/.claude-plugin/plugin.json',
 '.claude-plugin/marketplace.json',
 'plugins/framework-calidad/skills/quality-framework/SKILL.md',
 'plugins/framework-calidad/commands/quality-full.md',
 'plugins/framework-calidad/commands/quality-quick.md',
 'plugins/framework-calidad/commands/quality-security.md',
 'plugins/framework-calidad/commands/quality-deep-dive.md',
 'plugins/framework-calidad/skills/quality-framework/references/scoring.md'
];
let ok=true;
for(const f of must){ if(!fs.existsSync(f)){console.log('MISSING '+f);ok=false;} }
JSON.parse(fs.readFileSync('plugins/framework-calidad/.claude-plugin/plugin.json','utf8'));
JSON.parse(fs.readFileSync('.claude-plugin/marketplace.json','utf8'));
console.log(ok?'PLUGIN TREE OK':'PLUGIN TREE INCOMPLETE');
"
```
Expected: `PLUGIN TREE OK`

- [ ] **Step 3: Install the plugin in Claude Code (manual smoke test)**

In the Claude Code session run:
```
/plugin marketplace add .
/plugin install framework-calidad@framework-calidad-marketplace
```
Then confirm the four commands appear by typing `/quality` and checking the
autocomplete lists `quality-quick`, `quality-full`, `quality-security`, `quality-deep-dive`.
Expected: all four commands listed.

- [ ] **Step 4: End-to-end dry run**

Run `/quality-quick` against this repo (no path → cwd). Expected: the skill detects the
repo, inventories tools (most absent → logs 🧠 estimado), evaluates the §05/§06/§07/§10
subset, and writes `./quality-report/<today>/quick-report.md` with each finding marked
🔧 or 🧠. Verify the file exists:
```bash
ls quality-report/*/quick-report.md
```
Expected: a path is printed.

- [ ] **Step 5: Commit**

```bash
git add plugins/framework-calidad/README.md
git commit -m "docs(plugin): add README and usage; complete framework-calidad plugin"
```

---

## Self-Review Notes

- **Spec coverage:** estructura (Task 1–2), flujo híbrido 7 pasos (Task 3 SKILL.md), 4 modos + args (Tasks 4–7), salidas a `./quality-report/YYYY-MM-DD/` (SKILL.md step 6 + each command), errores/bordes (SKILL.md flow), manifest (Task 1), bundle autónomo (Task 2), transparencia de fuente (SKILL.md). Added marketplace.json + README (Tasks 1, 8) — required to actually install/use the plugin, within scope.
- **Out of scope respected:** no auto-install of tools, no score-only mode, no CI/CD, no history/diff.
- **Naming consistency:** plugin name `framework-calidad`, skill `quality-framework`, marketplace `framework-calidad-marketplace`, four commands `quality-{quick,full,security,deep-dive}` — used identically across all tasks and README.
- **Reference count check (Task 2):** 21 verbatim docs (00–18, 23, 24) + 4 friendly (herramientas, scoring, antipatrones, plantillas) = 25 files.
```
