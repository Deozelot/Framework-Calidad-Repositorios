# §04 — Cohesión y Acoplamiento

[← Volver al índice](INDEX.md) · Anterior: [§03](03-mantenibilidad.md) · Siguiente: [§05](05-buenas-practicas-codigo.md)

---

## Objetivo

Evaluar grado de independencia entre módulos/clases y cohesión interna de cada unidad.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 4.1 | LCOM (Lack of Cohesion of Methods) | NDepend / SonarQube |
| 4.2 | Imports cíclicos | `madge --circular` (TS), NetArchTest |
| 4.3 | Cross-module references | `grep -r "using OtherModule"` en cada módulo |
| 4.4 | Shared kernel real vs accidental | Tamaño + responsabilidades de `Shared/Common/` |
| 4.5 | DI registrada vs usada | Audit container registrations |
| 4.6 | Interface segregation | Tamaño interfaces (`I*.cs`) en métodos |
| 4.7 | Inyección por constructor vs field/property | Conteo `inject()` vs `[Inject]` vs constructor |

---

## Comandos de referencia

```bash
# Cíclicos TS
npx madge --circular --extensions ts src/

# Imports cruzados módulos .NET
for m in Auth Operations Produccion; do
  echo "=== $m ===";
  grep -rn "using Anh\.Gop\.\(Auth\|Operations\|Produccion\)\." \
    src/Modules/$m --include="*.cs" | grep -v "Anh\.Gop\.$m" | head;
done

# Tamaño de Shared
find src/Shared -name "*.cs" | wc -l
find src/Shared -name "*.cs" -exec wc -l {} \; | awk '{sum+=$1} END {print sum}'

# Interfaces > 10 métodos (fat interface)
grep -l "interface I" src --include="*.cs" -r | while read f; do
  m=$(grep -c "^\s*[A-Z].*(.*).*;\|^\s*Task" "$f");
  [ "$m" -gt 10 ] && echo "$f: $m methods";
done

# DI: registradas vs usadas
grep -rn "AddScoped\|AddTransient\|AddSingleton" src --include="*.cs" | wc -l
```

---

## Evidencia esperada

- Lista de imports cíclicos (debe ser vacía)
- Matriz cross-module: módulo A importa de módulo B (count)
- Lista de interfaces > 10 métodos
- Reporte LCOM por clase (top 10 peores)
- Ratio constructor injection vs property injection

---

## Indicadores

| Métrica | Verde | Rojo |
|---|---|---|
| Imports cíclicos | 0 | ≥1 |
| Módulos con imports cruzados | 0 (vía Shared) | Directos |
| LCOM máximo | <1 | >2 |
| Interfaz max methods | ≤7 | >12 |
| Shared kernel LOC | <10% del backend | >25% |
| DI registrations huérfanas | 0 | >5 |

---

## Anti-patrones

- Módulo A importa entidades de Módulo B directamente (debería ser vía DTO/contract)
- Shared kernel hipertrofiado con "utilidades" no compartidas realmente
- Interfaz "I<Service>" con 20+ métodos = Interface Pollution
- DbContext compartido entre bounded contexts
- Servicios singleton mutables
- Property injection (`[Inject] public IService Svc { get; set; }`) — oculta dependencias

---

## Refactor sugerido

| Síntoma | Refactor |
|---|---|
| Fat interface 20 métodos | Split por Concern (`IUserReader` + `IUserWriter`) |
| Cross-module import directo | Mover contrato a `Shared.Contracts` |
| Cíclico A↔B | Extract C que ambos consumen |
| Shared kernel grande | Mover utilidades a módulo dueño real |
| LCOM alto en clase | Split por responsabilidad |

---

## Referencias cruzadas

- Reglas dependencias arch → [§02 Arquitectura](02-arquitectura.md)
- Test arch enforcement → [§07 Testing](07-testing-cobertura.md)
- Interfaces fat → [§16 Backend](16-backend.md)
