# §05 — Buenas Prácticas de Código

[← Volver al índice](INDEX.md) · Anterior: [§04](04-cohesion-acoplamiento.md) · Siguiente: [§06](06-seguridad.md)

---

## Objetivo

Auditar adherencia a convenciones del stack: type safety, linting, formato, anti-patterns sintácticos.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 5.1 | Nullable enabled (`<Nullable>enable</Nullable>`) | `.csproj` inspection |
| 5.2 | TypeScript `strict: true` | `tsconfig.json` |
| 5.3 | Linting configurado (ESLint, StyleCop, Roslyn analyzers) | Archivos de config + CI |
| 5.4 | Formato auto-aplicado (Prettier, dotnet-format) | Hook pre-commit / CI |
| 5.5 | `any`/`object`/`dynamic` usage | `grep ": any"`, `grep "object\b"` |
| 5.6 | Empty catch blocks | `grep -r "catch.*{ *}"` |
| 5.7 | TODOs/FIXMEs/HACKs sin issue link | `grep -rn "TODO\|FIXME\|HACK"` |
| 5.8 | `console.log`/`Console.WriteLine` en producción | `grep` excluyendo tests |
| 5.9 | Magic numbers | Análisis estático SonarQube rule S109 |
| 5.10 | Naming consistente | Lint rule + convención documentada |
| 5.11 | Comentarios explican WHY no WHAT | Code review / heurística |
| 5.12 | XML/JSDoc en API pública | `<NoWarn>1591</NoWarn>` check |
| 5.13 | Inmutabilidad por defecto (records, readonly, const) | Audit |
| 5.14 | Async/await sin `.Result`/`.Wait()` | `grep -r "\.Result\|\.Wait()"` |
| 5.15 | `using` declarations / disposal patterns | Análisis estático |

---

## Comandos de referencia

```bash
# Nullable .NET
grep -rn "<Nullable>" --include="*.csproj" | grep -v enable

# TS strict
grep -A5 '"compilerOptions"' tsconfig.json | grep "strict"

# any types (TS)
grep -rn ": any\b\|<any>\|as any" src --include="*.ts" | grep -v ".spec.ts" | wc -l

# Empty catches
grep -rn "catch.*{\s*}" src --include="*.cs" --include="*.ts"

# TODOs/FIXMEs
grep -rn "TODO\|FIXME\|HACK\|XXX" src --include="*.cs" --include="*.ts" \
  | grep -v "\.spec\.\|\.test\."

# Console leftovers
grep -rn "Console\.WriteLine\|console\.log\|console\.error" src \
  --include="*.cs" --include="*.ts"

# .Result/.Wait() blocking calls
grep -rn "\.Result\b\|\.Wait()" src --include="*.cs" | grep -v "\.Result;"

# eslint/ts-ignore
grep -rn "@ts-ignore\|@ts-nocheck\|eslint-disable" src --include="*.ts" | wc -l
```

---

## Evidencia esperada

- Conteo `any` con archivos top
- Lista de TODOs sin issue link
- Output de `dotnet format --verify-no-changes` (debe ser exit 0)
- Output de `eslint .` (count errors/warnings)
- Conteo `.Result`/`.Wait()` blocking calls
- Tabla naming convention violations por archivo

---

## Umbrales

| Métrica | Verde | Ámbar | Rojo |
|---|---|---|---|
| `any` per 1000 LOC | 0 | 1-3 | >5 |
| TODOs sin link | 0 | 1-5 | >10 |
| Empty catches | 0 | 1 | >2 |
| `.Result`/`.Wait()` | 0 | 1-2 | >5 |
| Magic numbers | <5 | 5-20 | >50 |
| ESLint errors | 0 | 1-10 | >50 |

---

## Anti-patrones

- `catch (Exception ex) { /* silenced */ }`
- `as any` para callar el typechecker
- `// TODO: fix later` sin issue
- `Console.WriteLine("debug")` olvidado
- `await Task.FromResult(...)` envoltorio inútil
- `.Result` en async chain bloqueante (deadlock risk)
- Nombres `data`, `temp`, `value` sin contexto
- Comentarios `// increment i` (qué obvio) vs `// undo rounding bug R-042` (WHY)
- Magic numbers `if (count > 86400)` vs `const SECONDS_PER_DAY = 86400`

---

## Convenciones por stack

### .NET / C# 13
- `internal sealed` para clases no-API
- `private readonly` constructor injection
- `async/await` con `CancellationToken` siempre
- `record` para DTOs/Value Objects
- Strongly-typed IDs (`record UserId(Guid Value)`)
- XML `<summary>` en miembros públicos

### TypeScript / Angular
- `strict: true` + `noImplicitAny: true`
- Standalone components + `OnPush`
- `inject()` sobre constructor injection
- Signals para state local
- Mappers explícitos DTO ↔ Model

---

## Referencias cruzadas

- Documentación API → [§09 Documentación](09-documentacion.md)
- Linting en CI → [§08 CI/CD](08-cicd.md)
- Convenciones backend → [§16 Backend](16-backend.md)
- Convenciones frontend → [§15 Frontend](15-frontend.md)
