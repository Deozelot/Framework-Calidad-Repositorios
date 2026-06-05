# §07 — Testing y Cobertura

[← Volver al índice](INDEX.md) · Anterior: [§06](06-seguridad.md) · Siguiente: [§08](08-cicd.md)

---

## Objetivo

Medir presencia y calidad de tests en pirámide unit / integration / E2E, cobertura real, determinismo.

---

## Criterios

| # | Criterio | Cómo medir | Umbral |
|---|---|---|---|
| 7.1 | Tests unitarios presentes | Conteo `*Tests.cs`/`*.spec.ts` | Por handler/component |
| 7.2 | Cobertura líneas Domain+Application | `coverlet`, `vitest --coverage`, `nyc` | ≥80% / <60% |
| 7.3 | Cobertura branch | Mismas tools | ≥70% / <50% |
| 7.4 | Mutation testing | Stryker, Stryker.NET | Mutation score ≥60% |
| 7.5 | Tests integración (DB, HTTP) | `*IntegrationTests.cs` | Por endpoint crítico |
| 7.6 | Tests E2E (Playwright/Cypress) | Folder `e2e/` + CI | Flujos críticos cubiertos |
| 7.7 | Architecture tests (NetArchTest, ArchUnit) | Conteo `[Fact]` en `*ArchTests` | ≥1 regla por capa |
| 7.8 | Tests deterministas (sin DateTime.Now, sleep) | `grep DateTime.Now\|Sleep` en tests | 0 / >5 |
| 7.9 | Tests rápidos | `dotnet test` tiempo | <30s unit / >2min |
| 7.10 | Tests flaky | Retry count en CI | 0% / >5% |
| 7.11 | Tests skip/ignore mergeados | `grep -r "\[Skip\]\|xit\|xdescribe"` | 0 / >0 |
| 7.12 | Reports coverage publicados en CI | Codecov/Coveralls/SonarCloud badge | Sí / No |
| 7.13 | Naming convention tests | `<Method>_<Scenario>_<Expected>` audit | Consistente / Mezclado |
| 7.14 | Test pyramid balance | Ratio Unit/Integration/E2E | 70/20/10 / Invertido |

---

## Comandos de referencia

```bash
# Backend: coverage con coverlet
dotnet test --collect:"XPlat Code Coverage" \
  /p:CoverletOutputFormat=cobertura,opencover \
  /p:Threshold=80 /p:ThresholdType=line

# Generar HTML report
dotnet tool install -g dotnet-reportgenerator-globaltool
reportgenerator -reports:"**/coverage.cobertura.xml" \
  -targetdir:"coverage-report" -reporttypes:Html

# Frontend: vitest coverage
npx vitest run --coverage --reporter=verbose

# E2E Playwright
npx playwright test --reporter=html

# Mutation testing .NET
dotnet tool install -g dotnet-stryker
dotnet stryker

# Mutation testing TS
npx stryker run

# Test count por tipo
find src -name "*Tests.cs" -path "*Tests.Unit*" | wc -l        # Unit
find src -name "*Tests.cs" -path "*Tests.Integration*" | wc -l  # Integration
find e2e -name "*.spec.ts" | wc -l                              # E2E

# Tests no deterministas
grep -rn "DateTime\.Now\|DateTime\.UtcNow\|Thread\.Sleep\|setTimeout" \
  --include="*Tests*.cs" --include="*.spec.ts"

# Tests skipped
grep -rn "\[Skip\|\[Fact(Skip\|xit(\|xdescribe(\|\.skip(" \
  --include="*Tests*.cs" --include="*.spec.ts"
```

---

## Evidencia esperada

- Cobertura % por capa (Domain / Application / Infrastructure / Api)
- HTML report adjunto
- Pyramid count (unit / integration / E2E)
- Tiempo total ejecución suite
- Lista de tests skipped (debe ser vacía)
- Lista de tests con `DateTime.Now` directo (debe ser vacía)

---

## Umbrales cobertura por capa

| Capa | Verde | Ámbar | Rojo |
|---|---|---|---|
| Domain | ≥90% | 80-89% | <80% |
| Application (handlers) | ≥85% | 70-84% | <70% |
| Infrastructure | ≥60% | 40-59% | <40% |
| Api (controllers) | ≥70% (via integration) | 50-69% | <50% |

## Pirámide ideal

```
        /\           E2E (10%)
       /  \          ~5-20 escenarios críticos
      /----\
     /      \        Integration (20%)
    /        \       ~1 por endpoint crítico
   /----------\
  /            \     Unit (70%)
 /              \    1+ por handler/service
/________________\
```

**Pirámide invertida (anti-patrón):** muchos E2E, pocos unit. Frágil, lento, difícil debug.

---

## Anti-patrones

- Tests con `[Skip]` mergeados a main
- `DateTime.UtcNow` directo en assertion (no determinista)
- Tests que dependen de orden de ejecución
- `Thread.Sleep(5000)` en lugar de `await Task.Delay` con `Microsoft.Extensions.TimeProvider`
- Snapshot tests sin revisión (todo se acepta automáticamente)
- E2E que toca BD real de QA y deja datos basura
- Test "happy path" único sin edge cases
- Test que pasa porque no aserción real (`Assert.True(true)`)
- Cobertura alta por archivos auto-generados (excluir migrations, DTOs)
- Tests de UI que verifican CSS literal (frágil)

---

## Estrategias para mejorar cobertura

1. **Baseline:** medir hoy. Definir target +5%/sprint.
2. **PR gate:** bloquear si cobertura del cambio < umbral.
3. **Priorizar handlers de negocio** sobre DTOs/configurations.
4. **TimeProvider** para reglas con fechas (caducidad, plazos).
5. **Test data builders** (`new WellBuilder().WithName("X").Build()`).
6. **In-memory DB** (Sqlite/EF InMemory) para tests integración rápidos.
7. **WireMock** para mocks HTTP en integration tests.

---

## Referencias cruzadas

- Architecture tests → [§02 Arquitectura](02-arquitectura.md)
- E2E en CI → [§08 CI/CD](08-cicd.md)
- Performance benchmarks → [§12 Performance](12-performance.md)
