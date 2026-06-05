# §13 — Dependencias y Supply Chain

[← Volver al índice](INDEX.md) · Anterior: [§12](12-performance.md) · Siguiente: [§14](14-datos-persistencia.md)

---

## Objetivo

Evaluar gestión de paquetes, vulnerabilidades, frescura, supply chain security.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 13.1 | Lockfiles committeados | `package-lock.json`, `packages.lock.json` |
| 13.2 | Versiones pinned o ranges acotados | `^X.Y.Z` vs `*` |
| 13.3 | Dependabot/Renovate activo | `.github/dependabot.yml` |
| 13.4 | Dependencias outdated | `npm outdated`, `dotnet list package --outdated` |
| 13.5 | Dependencias deprecated | Warnings en install |
| 13.6 | Licencias compatibles | License audit (FOSSA, license-checker) |
| 13.7 | SBOM generado | CycloneDX / SPDX en releases |
| 13.8 | Verificación de paquetes (npm provenance, NuGet signed) | Config |
| 13.9 | Mirror interno (artifact registry) | Si aplica organización |
| 13.10 | Vendor lockin documentado | Audit servicios cloud específicos |

---

## Comandos de referencia

```bash
# Lockfiles presentes
ls package-lock.json packages.lock.json yarn.lock pnpm-lock.yaml 2>/dev/null

# Outdated .NET
dotnet list package --outdated --include-transitive
dotnet list package --deprecated

# Outdated Node
npm outdated --long
npm audit --json

# License check
npx license-checker --summary
npx license-checker --excludePackages 'react@^18.0.0'

# SBOM generation
npm install -g @cyclonedx/cdxgen
cdxgen -o sbom.json .

# Versiones pinned vs ranges
cat package.json | jq '.dependencies, .devDependencies' \
  | grep -oE '"\^[0-9]|"~[0-9]|"\*"|">=' | sort | uniq -c

# Dependencias usadas vs no usadas (TS)
npx depcheck

# Paquetes con vulnerabilidades known
trivy fs .
osv-scanner -r .
```

---

## Evidencia esperada

- Lista lockfiles presentes
- Reporte outdated por severidad (major/minor/patch)
- Lista deprecated packages
- Reporte license check (problemas con GPL en proyecto comercial)
- SBOM CycloneDX/SPDX
- Reporte vulnerabilidades por CVSS score
- Lista paquetes huérfanos (depcheck)

---

## Umbrales

| Métrica | Verde | Ámbar | Rojo |
|---|---|---|---|
| Vulnerabilidades CRITICAL | 0 | — | ≥1 |
| Vulnerabilidades HIGH | 0 | 1-3 | >3 |
| Vulnerabilidades MEDIUM | <5 | 5-15 | >15 |
| Paquetes major outdated | <5 | 5-15 | >15 |
| Paquetes deprecated | 0 | 1-3 | >3 |
| Licencias incompatibles | 0 | — | ≥1 |
| Lockfile commiteado | Sí | — | No |
| Dependabot activo | Sí | — | No |

---

## Anti-patrones

- `"react": "*"` o `"latest"` — version drift garantizado
- Sin lockfile committeado — builds no reproducibles
- `npm install --force` para "callar" peer dependency warnings
- Dependencias prod en `devDependencies` o viceversa
- Bundle frontend importa moment.js (deprecated) en lugar de date-fns/dayjs
- Lodash full package cuando solo se usan 2 funciones (vs `lodash.<fn>`)
- Paquetes con last publish > 2 años (abandono)
- Single-maintainer packages críticos (left-pad bus factor)
- Sin Dependabot → vulnerabilidades acumulan
- Mirror interno desactualizado vs origin

---

## Supply chain security

| Práctica | Tool |
|---|---|
| Verify package signatures | npm provenance, NuGet signed packages |
| Lock by hash | npm `--package-lock-only`, `--integrity` |
| Scan dependencies (SAST) | Snyk, Dependabot, Sonatype |
| SBOM en cada release | CycloneDX (`cdxgen`), SPDX |
| Audit transitive deps | `npm audit`, `dotnet list --include-transitive` |
| Block install with vulns | `--audit-level=high` |
| Verify supplier reputation | npm package weekly downloads + maintainer history |
| Pin Docker base images by digest | `FROM image@sha256:...` |
| Scan container images | Trivy, Grype, Snyk Container |

---

## Versioning strategy

| Modelo | Cuándo |
|---|---|
| Exact pin (`1.2.3`) | Máxima reproducibilidad, requiere update manual frecuente |
| Caret (`^1.2.3`) | Minor + patch auto, mayor bug risk |
| Tilde (`~1.2.3`) | Solo patch auto, conservador |
| Range (`>=1.2.0 <2.0.0`) | Flexibilidad transitiva |
| `latest`/`*` | NUNCA en producción |

Recomendado: **caret en dev/CI + lockfile committed → reproducibilidad garantizada**.

---

## Dependency hygiene mensual

```bash
# 1. Audit
npm audit
dotnet list package --vulnerable

# 2. Outdated review
npm outdated
dotnet list package --outdated

# 3. Update PR (patch/minor automático con Dependabot)
# 4. Major updates revisar changelog + test E2E
# 5. Remove unused
npx depcheck
```

---

## Referencias cruzadas

- Vulnerabilidades CVE → [§06 Seguridad](06-seguridad.md)
- Dependabot config → [§08 CI/CD](08-cicd.md)
- Bundle size impact → [§12 Performance](12-performance.md)
