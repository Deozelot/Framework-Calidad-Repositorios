# §09 — Documentación

[← Volver al índice](INDEX.md) · Anterior: [§08](08-cicd.md) · Siguiente: [§10](10-procesos-trazabilidad.md)

---

## Objetivo

Auditar completitud y vigencia de documentación interna y externa del proyecto.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 9.1 | README quickstart funcional | Validar pasos manualmente |
| 9.2 | Arquitectura documentada (C4, diagrama bloques) | Existencia + fecha < 3 meses |
| 9.3 | API documentada (OpenAPI/Swagger) | Endpoint `/swagger`, `/scalar` accesible |
| 9.4 | XML/JSDoc en API pública | Coverage por endpoint |
| 9.5 | ADRs registrados | Conteo `docs/adr/` |
| 9.6 | Runbook operacional | `docs/runbook.md` |
| 9.7 | Contributing guide | `CONTRIBUTING.md` |
| 9.8 | Onboarding doc | `ONBOARDING.md` o equivalente |
| 9.9 | Glosario del dominio | `docs/glossary.md` |
| 9.10 | Changelog | `CHANGELOG.md` actualizado |
| 9.11 | Convenciones de commit documentadas | Conventional Commits + linter |
| 9.12 | Variables de entorno documentadas | `.env.example` con comentarios |
| 9.13 | Diagrams as code | PlantUML / Mermaid en repo |
| 9.14 | Storybook / Component catalog | Build verde + URL desplegada |

---

## Comandos de referencia

```bash
# Archivos de gobernanza
ls README.md CONTRIBUTING.md CODE_OF_CONDUCT.md LICENSE SECURITY.md CHANGELOG.md 2>/dev/null

# ADRs count
find docs/adr specs/*/adr -name "*.md" 2>/dev/null | wc -l

# Última actualización docs
find docs -name "*.md" -exec stat --format="%Y %n" {} \; | sort -rn | head

# Coverage XML docs .NET
grep -c "<summary>" src/**/*Controller.cs

# Swagger endpoint reachable
curl -sf http://localhost:5000/swagger/v1/swagger.json | jq '.paths | length'

# Storybook build
npm run build-storybook && echo "OK" || echo "FAIL"

# .env.example presente y comentado
test -f .env.example && wc -l .env.example
```

---

## Evidencia esperada

- Lista de archivos gobernanza presentes/ausentes
- Fecha última actualización por documento clave
- Conteo ADRs por estado (Proposed/Accepted/Deprecated)
- Coverage Swagger (% endpoints con `<summary>`)
- URL Storybook desplegado (si aplica)
- Diff `.env.example` vs variables usadas en código

---

## Documentos imprescindibles

| Documento | Audiencia | Contenido mínimo |
|---|---|---|
| `README.md` | Todos | Qué hace + quickstart + links principales |
| `CONTRIBUTING.md` | Devs externos/internos | Branching + commits + PR process |
| `ARCHITECTURE.md` o `docs/architecture/` | Devs/Arquitectos | C4 nivel 1+2, decisiones |
| `docs/adr/` | Arquitectos | ADRs numerados secuencial |
| `CHANGELOG.md` | Stakeholders | Semver + cambios por versión |
| `SECURITY.md` | Pentesters | Cómo reportar vulnerabilidades |
| `CODE_OF_CONDUCT.md` | Comunidad | Comportamiento esperado |
| `LICENSE` | Legal | Licencia explícita |
| `.env.example` | Devs | Variables sin valores reales |
| `docs/runbook.md` | Ops | Procedimientos operacionales |
| `docs/glossary.md` | Todos | Vocabulario del dominio |
| `ONBOARDING.md` | Devs nuevos | Setup local + arquitectura + primeros pasos |

---

## Umbrales

| Métrica | Verde | Rojo |
|---|---|---|
| README último update | <30d | >180d |
| Quickstart funciona en clean checkout | Sí | No |
| ADRs registrados | ≥1 por feature compleja | 0 |
| OpenAPI completo (todos endpoints) | 100% | <80% |
| XML `<summary>` en API pública | ≥80% | <30% |
| Storybook stories por componente UIKit | ≥1 | 0 |
| Variables `.env` documentadas | 100% | <80% |

---

## Anti-patrones

- README "Project X — TODO: write docs"
- ADRs como issues de GitHub (se pierden con tiempo)
- OpenAPI con `description: "TODO"`
- Quickstart que asume entorno preconfigurado del autor
- Doc dice "ejecutar `make build`" pero `Makefile` no existe
- Diagramas como `.png` no editables vs `.puml`/`.mmd` editables
- Storybook con build roto en main
- CHANGELOG auto-generado sin curaduría
- Comments en código que duplican lo obvio
- `.env.example` con secrets reales (sí, sucede)

---

## Sugerencias prácticas

- **Docs-as-code:** versionar docs en mismo repo que código
- **ADR template:** título + contexto + decisión + consecuencias + estado
- **Mermaid en GitHub:** se renderiza nativo, mejor que `.png` no editables
- **Conventional Commits:** auto-genera changelog
- **Storybook + Chromatic:** docs visuales con regresión
- **OpenAPI-first:** definir contrato YAML antes que código
- **Swashbuckle/Scalar:** auto-doc desde XML comments
- **Compodoc (Angular):** auto-doc de componentes
- **MkDocs/Docusaurus:** site portable de docs

---

## Referencias cruzadas

- OpenAPI versionado → [§16 Backend](16-backend.md)
- Storybook UI components → [§15 Frontend](15-frontend.md)
- ADRs vs decisiones → [§02 Arquitectura](02-arquitectura.md)
- CHANGELOG + commits → [§10 Procesos](10-procesos-trazabilidad.md)
