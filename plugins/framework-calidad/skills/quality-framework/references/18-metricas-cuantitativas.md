# §18 — Métricas Cuantitativas Recomendadas

[← Volver al índice](INDEX.md) · Anterior: [§17](17-deuda-tecnica.md) · Siguiente: [§19](19-herramientas.md)

---

## Objetivo

Catálogo de métricas con comando ejecutable y frecuencia sugerida. Para alimentar dashboards y comparativas temporales.

---

## Tabla maestra

| Métrica | Comando ejemplo | Frecuencia |
|---|---|---|
| LOC totales | `find . -name "*.cs" -o -name "*.ts" \| xargs wc -l` | Semanal |
| Archivos > 500 LOC | `find . -name "*.cs" -exec wc -l {} \; \| awk '$1>500'` | Sprint |
| Commits/semana por autor | `git shortlog -sn --since=1.week` | Semanal |
| PRs abiertos > 7 días | GitHub API | Diaria |
| Tests fallidos en main | CI history | Diaria |
| Cobertura por módulo | Coverlet/nyc report | PR |
| Vulnerabilidades por severidad | `npm audit` + `dotnet list --vulnerable` | Semanal |
| Build time | CI logs avg last 10 builds | Sprint |
| Lead time PR | GitHub insights | Sprint |
| Deploy frequency | Tags/releases por semana | Sprint |

---

## Bloques de métricas — Por dimensión

### Tamaño y crecimiento del repo

```bash
# LOC por lenguaje (cloc requerido)
cloc . --exclude-dir=node_modules,bin,obj,dist,publish,.git

# Archivos por tipo
find . -type f \( -name "*.cs" -o -name "*.ts" -o -name "*.tsx" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" \
  | sed 's/.*\.//' | sort | uniq -c

# Crecimiento histórico (cada commit)
git log --pretty=format:"%h %ad" --date=short --reverse \
  | while read hash date; do
      loc=$(git show "$hash" --stat | tail -1 | awk '{print $4-$6}');
      echo "$date $loc";
    done | tail -30

# Top 20 archivos más grandes
find . -type f \( -name "*.cs" -o -name "*.ts" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" \
  -not -path "*/bin/*" -not -path "*/obj/*" \
  -exec wc -l {} \; | sort -rn | head -20
```

### Actividad y velocidad

```bash
# Commits/día último mes
git log --since="1 month ago" --pretty=format:"%ad" --date=short \
  | sort | uniq -c

# Commits por autor último trimestre
git shortlog -sn --since="3 months ago"

# PRs mergeados último mes
gh pr list --state merged --search "merged:>$(date -d '1 month ago' +%Y-%m-%d)" \
  --limit 100 | wc -l

# Tiempo promedio PR open→merge (horas)
gh pr list --state merged --limit 100 \
  --json createdAt,mergedAt \
  | jq '[.[] | ((.mergedAt | fromdateiso8601) - (.createdAt | fromdateiso8601))/3600] | add/length'

# PRs abiertos > 7 días
gh pr list --state open --search "created:<$(date -d '7 days ago' +%Y-%m-%d)" \
  | wc -l

# Tags por mes (deploy proxy)
git tag --sort=-creatordate | head -20

# Lead time medio (DORA)
# Tiempo entre primer commit feature branch → merge a main
```

### Calidad

```bash
# Cobertura porcentaje (después de ejecutar tests)
coverlet bin/Debug/MyApp.dll --target dotnet --targetargs "test --no-build" \
  --format opencover --threshold 80

# Code smells (SonarQube CLI)
sonar-scanner -Dsonar.projectKey=myproject

# Duplicación
npx jscpd src/ --threshold 3

# Dependencias outdated
npm outdated --json | jq 'length'
dotnet list package --outdated | grep ">" | wc -l

# Vulnerabilidades por severidad
npm audit --json | jq '.metadata.vulnerabilities'

# Secrets en historia
gitleaks detect --source . --log-opts="--all" --report-format json \
  | jq 'length'
```

### Tests

```bash
# Total tests
find . -name "*Tests.cs" -not -path "*/bin/*" -not -path "*/obj/*" \
  | xargs grep -l "\[Fact\]\|\[Theory\]" | wc -l

# Tests skipped
grep -rn "\[Fact(Skip\|\[Skip\|xit(\|\.skip(" \
  --include="*Tests.cs" --include="*.spec.ts" | wc -l

# Tiempo total suite
dotnet test --logger "console;verbosity=minimal" 2>&1 \
  | grep "Total time:"

# Cobertura por capa (después de generar report)
reportgenerator -reports:coverage.cobertura.xml -targetdir:report \
  -reporttypes:JsonSummary

cat report/Summary.json | jq '.coverage'
```

### Hot files / Churn

```bash
# Files más modificados (6 meses)
git log --since="6 months ago" --pretty=format: --name-only \
  | grep -E "\.(cs|ts)$" | sort | uniq -c | sort -rn | head -20

# Files que crecen rápido
for f in $(git log --since="6 months ago" --pretty=format: --name-only \
  | sort -u | grep -E "\.(cs|ts)$"); do
    [ -f "$f" ] || continue;
    growth=$(git log --since="6 months ago" --follow --pretty=tformat: \
      --numstat "$f" | awk '{add+=$1; del+=$2} END {print add-del}');
    [ "$growth" -gt 200 ] && echo "$growth $f";
done | sort -rn | head -10

# Co-cambio (archivos modificados juntos)
git log --since="6 months ago" --name-only --pretty=format:'==COMMIT==' \
  | awk 'BEGIN{RS="==COMMIT==\n"} {
      n=split($0, files, "\n");
      for (i=1; i<=n; i++) for (j=i+1; j<=n; j++) {
        if (files[i] && files[j]) print files[i]" "files[j];
      }}' | sort | uniq -c | sort -rn | head -10
```

---

## Frecuencias recomendadas

| Cadencia | Métricas |
|---|---|
| **Por PR** | Cobertura del diff, build time, tests fallidos del cambio |
| **Diaria** | PRs abiertos > 7d, CI build success rate, alerts críticas |
| **Semanal** | Commits/autor, LOC totales, vulnerabilidades, dep outdated |
| **Sprint (2 sem)** | Velocity, lead time, tiempo build avg, files > 500 LOC, hot files |
| **Mensual** | Bus factor, coverage trend, deuda técnica score, DORA metrics |
| **Trimestral** | Framework completo, refactor planning |
| **Anual** | Penetration test, license audit, dependency major upgrades |

---

## Dashboard sugerido

| Cuadrante | Métricas |
|---|---|
| **Flujo (DORA)** | Deploy freq, Lead time, Change failure rate, MTTR |
| **Calidad** | Coverage, # tests, bugs abiertos, code smells |
| **Velocidad** | Commits/dev, PRs/sprint, lead time PR |
| **Riesgo** | Vulnerabilidades, dep outdated, secrets findings |

---

## Para reportes ejecutivos — KPIs Top-Level

| KPI | Cómo medir |
|---|---|
| **Velocity** | Story points o PRs completados por sprint |
| **Predictability** | % stories completadas vs planeadas |
| **Quality** | Bugs/sprint, escape rate (bugs llegando a prod) |
| **Reliability** | Uptime %, MTTR, error rate |
| **Velocity change** | Tendencia 6 sprints |
| **Tech debt %** | Tiempo dedicado a deuda / total |

---

## Comparativas temporales

Para tracking, mantener snapshot semanal en CSV/JSON:

```csv
date,total_loc,test_count,coverage_pct,build_time_min,vulns_high,prs_merged,bus_factor
2026-05-01,12000,200,72.5,8,2,15,3
2026-05-08,12450,210,74.2,8,1,18,3
2026-05-15,13100,215,75.8,9,1,22,3
...
```

Genera gráficos trend para detectar regresiones tempranas.

---

## Referencias cruzadas

- LOC por módulo → [§03 Mantenibilidad](03-mantenibilidad.md)
- Bus factor / hot files → [§17 Deuda](17-deuda-tecnica.md)
- Build time CI → [§08 CI/CD](08-cicd.md)
- DORA metrics → [§10 Procesos](10-procesos-trazabilidad.md)
