# §02 — Arquitectura

[← Volver al índice](INDEX.md) · Anterior: [§01](01-estructura-gobernanza.md) · Siguiente: [§03](03-mantenibilidad.md)

---

## Objetivo

Evaluar topología, separación de capas, bounded contexts, patrones aplicados y enforcement automático de reglas arquitectónicas.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 2.1 | Topología documentada (monolito modular / microservicios / etc.) | Diagrama C4 o equivalente en `docs/architecture/` |
| 2.2 | Capas (Domain/Application/Infrastructure/Api) explícitas | Convención + estructura de proyectos |
| 2.3 | Regla de dependencias enforced en CI | `NetArchTest` (.NET), `archunit-net`, `dependency-cruiser` (TS) |
| 2.4 | Bounded contexts delimitados | 1 dir = 1 contexto. Verificar overlap con grep cruzado |
| 2.5 | Patrón CQRS / MVC / Onion declarado y aplicado | Inspección coherencia handlers/controllers |
| 2.6 | Acoplamiento entre módulos | `grep` de imports cruzados Module A→B |
| 2.7 | Inversión de dependencias (interfaces en capa superior) | Conteo interfaces `I*` vs implementaciones |
| 2.8 | Patrones Hexagonal / Ports & Adapters en integraciones | Estructura `Ports/` + `Adapters/` |
| 2.9 | ADRs por decisión significativa | `docs/adr/NNN-*.md` |
| 2.10 | Diagrama de despliegue actualizado | `docs/deployment.md` con fecha < 3 meses |

---

## Comandos de referencia

```bash
# Capas (.NET)
find src -name "*.csproj" | sed 's|.*\.\(Domain\|Application\|Infrastructure\|Api\)\.csproj|\1|' | sort | uniq -c

# Imports cruzados módulo (.NET)
grep -rn "using Anh\.Gop\.<OtroModulo>" src/Modules/<MiModulo> --include="*.cs"

# Imports cíclicos (TS)
npx madge --circular src/

# Dependency rules (.NET)
dotnet test tests/Anh.Gop.Architecture.Tests

# Conteo handlers CQRS
find src -name "*Handler.cs" -not -path "*/Tests*" | wc -l
find src -name "*Controller.cs" -not -path "*/Tests*" | wc -l

# Conteo ADRs
find docs/adr specs/*/adr -name "*.md" 2>/dev/null | wc -l
```

---

## Evidencia esperada

- Diagrama topología (C4 nivel 1+2)
- Tabla LOC + archivos por capa por módulo
- Resultado tests arquitecturales (verde/rojo + reglas activas)
- Lista de ADRs registrados
- Matriz acoplamiento módulo×módulo

---

## Patrones típicos a verificar

| Patrón | Señal de cumplimiento |
|---|---|
| Clean Architecture | Domain no depende de Infrastructure (test arch) |
| CQRS | Commands separados de Queries, dispatcher dedicado |
| Hexagonal | Carpetas `Ports/` + `Adapters/` en integraciones |
| Repository pattern | Interfaz `I<Entity>Repository` en Application, impl en Infrastructure |
| Unit of Work | `IUnitOfWork` con `CommitAsync()` |
| Domain Events | `*Event.cs` + dispatcher |
| Strongly-typed IDs | `record <Entity>Id` en lugar de `Guid` puro |

---

## Anti-patrones

- Module A importa internals de Module B directamente
- Domain importa EF Core / ASP.NET
- Application importa Infrastructure
- Controller con lógica de negocio (acceso DbContext directo)
- ADRs ausentes pero decisiones arquitectónicas tomadas
- Patrón declarado (CQRS) pero handlers usan repositorios mezclados con DbContext

---

## Referencias cruzadas

- Tamaño dentro de capa → [§03 Mantenibilidad](03-mantenibilidad.md)
- Multitenancy enforcement → [§14 Datos](14-datos-persistencia.md)
- Tests arquitecturales → [§07 Testing](07-testing-cobertura.md)
