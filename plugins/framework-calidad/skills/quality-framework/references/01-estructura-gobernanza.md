# §01 — Estructura y Gobernanza del Repo

[← Volver al índice](INDEX.md) · Anterior: [§00](00-como-usar.md) · Siguiente: [§02](02-arquitectura.md)

---

## Objetivo

Evaluar layout raíz, convenciones de naming, presencia de archivos de gobernanza y limpieza del repo.

---

## Criterios

| # | Criterio | Cómo medir | Verde / Rojo |
|---|---|---|---|
| 1.1 | Layout raíz coherente | Inspección `ls -la` + comparación contra convención del stack | ≤15 dirs / >25 dirs |
| 1.2 | Convenciones de naming uniformes | Regex sobre carpetas/archivos por tipo | 1 patrón / 3+ patrones |
| 1.3 | Profundidad máxima de carpetas | `find . -type d -maxdepth N` | ≤6 / >9 niveles |
| 1.4 | Artefactos build committeados | `du -sh` + listar `.dll/.pdb/dist/publish` | 0MB / >10MB |
| 1.5 | `.gitignore` completo y aplicado | Comparar `.gitignore` vs `git ls-files` | 0 inconsistencias / >5 |
| 1.6 | README raíz presente y útil | Existencia + secciones (quickstart, arch, contributing) | Sí / Ausente |
| 1.7 | LICENSE presente | Archivo en raíz | Sí / No |
| 1.8 | Documentación de contribución | `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md` | Sí / No |
| 1.9 | Colisión de IDs en trazabilidad/specs | Diff de prefijos numéricos | 0 / ≥1 |
| 1.10 | Carpetas huérfanas (sin enrutar/usar) | Inspección de referencias | 0 / >2 |

---

## Comandos de referencia

```bash
# Layout raíz
ls -la
find . -maxdepth 2 -type d -not -path "*/.git*" -not -path "*/node_modules*" | sort

# Tamaño de carpetas sospechosas
du -sh publish/ dist/ build/ 2>/dev/null

# Profundidad
find . -type d | awk -F/ '{print NF}' | sort -rn | head -1

# Archivos > umbral en raíz
find . -maxdepth 1 -type f -size +1M

# Colisión IDs (ejemplo HU-NNN)
ls Trazabilidad/ | awk -F- '{print $1"-"$2}' | sort | uniq -d
```

---

## Evidencia esperada

- Output de `ls -la` raíz
- Tabla con tamaño/profundidad por carpeta de alto nivel
- Lista de archivos committeados en `.gitignore` (si los hay)
- Lista de archivos de gobernanza presentes/ausentes (README, LICENSE, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY)

---

## Anti-patrones inmediatos

- `publish/`, `dist/`, `node_modules/`, `bin/`, `obj/` versionados
- `.env*` con secrets en commit
- Múltiples convenciones de naming en mismo nivel (`HU_GOP_06_*` + `Forma_204_*`)
- 25+ dirs en raíz sin agrupación
- README ausente o "Hello World" sin actualizar

---

## Referencias cruzadas

- Convenciones naming archivos código → [§05 Buenas prácticas](05-buenas-practicas-codigo.md)
- Archivos de gobernanza → [§09 Documentación](09-documentacion.md)
- Branch protection / CODEOWNERS → [§08 CI/CD](08-cicd.md)
