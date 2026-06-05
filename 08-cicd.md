# §08 — CI/CD y Automatización

[← Volver al índice](INDEX.md) · Anterior: [§07](07-testing-cobertura.md) · Siguiente: [§09](09-documentacion.md)

---

## Objetivo

Evaluar workflows automatizados de build/test/deploy, branch protection, gates de calidad.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 8.1 | CI workflows definidos | `.github/workflows/`, `.gitlab-ci.yml`, `azure-pipelines.yml` |
| 8.2 | Build verde en main | Badge / status última semana |
| 8.3 | Tests ejecutados en cada PR | Required check |
| 8.4 | Coverage publicada por PR | Comment bot |
| 8.5 | Linting en CI | Step en workflow |
| 8.6 | Branch protection main/develop | GitHub settings |
| 8.7 | Required reviews (≥1, ≥2) | Branch protection |
| 8.8 | Dismiss stale reviews | Branch protection |
| 8.9 | CODEOWNERS configurado | `.github/CODEOWNERS` |
| 8.10 | Deploy automatizado a staging | Pipeline trigger en merge develop |
| 8.11 | Deploy manual a producción con approval | Environment protection |
| 8.12 | Rollback strategy documentada | Runbook |
| 8.13 | Feature flags (LaunchDarkly, GrowthBook) | Servicio + uso en código |
| 8.14 | Tiempo build CI | <10min / >30min |
| 8.15 | Caching de dependencias | npm/nuget cache en workflow |

---

## Comandos de referencia

```bash
# Listar workflows
ls .github/workflows/

# Status últimos 20 runs main
gh run list --branch main --limit 20

# Branch protection rules
gh api repos/:owner/:repo/branches/main/protection

# CODEOWNERS
cat .github/CODEOWNERS

# Pipeline build time avg
gh run list --branch main --limit 50 --json conclusion,startedAt,updatedAt \
  | jq '[.[] | select(.conclusion=="success") |
    ((.updatedAt | fromdateiso8601) - (.startedAt | fromdateiso8601))] | add/length/60'

# Required checks failing
gh pr list --json statusCheckRollup

# Workflows tiempo paso
gh run view <run-id> --log
```

---

## Evidencia esperada

- Lista workflows + trigger (push/PR/schedule/dispatch)
- Tasa de build verde últimos 30 días
- Tiempo build promedio (avg, p95)
- Branch protection rules screenshot/JSON
- Lista CODEOWNERS
- Environment protection rules para production

---

## Umbrales

| Métrica | Verde | Ámbar | Rojo |
|---|---|---|---|
| Build success rate main (30d) | ≥95% | 85-94% | <85% |
| Build time avg | <10min | 10-20min | >30min |
| PR review approval req | ≥1 | 0 | — |
| Required checks | tests + lint + build | solo build | ninguno |
| Auto-deploy a staging | sí | manual | no |
| Production deploy approval | required | optional | no |
| Cache hit rate deps | >80% | 50-80% | <50% |

---

## Pipeline stages mínimo recomendado

```yaml
# .github/workflows/ci.yml
jobs:
  lint:
    # ESLint + Prettier + dotnet-format --verify-no-changes
  build:
    needs: lint
    # dotnet build + ng build
  test-unit:
    needs: build
    # dotnet test --filter Category=Unit + vitest
  test-integration:
    needs: build
    # dotnet test --filter Category=Integration (con SQL test container)
  coverage:
    needs: [test-unit, test-integration]
    # coverlet + reportgenerator + upload Codecov
  security:
    needs: build
    # gitleaks + trivy + dotnet list --vulnerable + npm audit
  arch-tests:
    needs: build
    # NetArchTest suite
  e2e:
    needs: build
    # Playwright contra ambiente de QA
```

---

## Anti-patrones

- Workflow que solo corre en push a main (no en PR)
- Tests opcionales / no required check
- Build pasa porque tests están skipped silenciosamente
- Deploy a prod sin approval gate
- Sin rollback automatizado
- Cache desactivado → builds 30min innecesarios
- Sin CODEOWNERS → cualquiera mergea cualquier cosa
- Force push permitido en main
- Pipeline secrets en variables de workflow (no en GitHub Secrets)
- Tests E2E que dependen de datos seed manuales

---

## Branch protection recomendado (main/develop)

- ✅ Require pull request before merging
- ✅ Require approvals (≥1 para develop, ≥2 para main)
- ✅ Dismiss stale approvals when new commits pushed
- ✅ Require review from CODEOWNERS
- ✅ Require status checks to pass:
  - lint
  - build
  - test-unit
  - test-integration
  - coverage (threshold ≥80%)
  - security-scan
  - arch-tests
- ✅ Require branches to be up to date
- ✅ Require linear history (opcional)
- ✅ Require signed commits (regulado)
- ✅ Include administrators
- ❌ Allow force pushes (NUNCA)
- ❌ Allow deletions (NUNCA)

---

## Deployment strategy

| Estrategia | Cuándo |
|---|---|
| **Rolling update** | Cambios backwards-compatible, default |
| **Blue/Green** | Cambios riesgosos, rollback rápido |
| **Canary** | Features grandes, rollout por % |
| **Feature flags** | Activar/desactivar sin redeploy |

---

## Referencias cruzadas

- Tests required → [§07 Testing](07-testing-cobertura.md)
- Secrets scan → [§06 Seguridad](06-seguridad.md)
- Deploy observability → [§11 Observabilidad](11-observabilidad.md)
- Convención commits → [§10 Procesos](10-procesos-trazabilidad.md)
