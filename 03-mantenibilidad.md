# §03 — Mantenibilidad: Tamaño y Cohesión

[← Volver al índice](INDEX.md) · Anterior: [§02](02-arquitectura.md) · Siguiente: [§04](04-cohesion-acoplamiento.md)

---

## Objetivo

Medir tamaño físico del código y violaciones de SRP que correlacionan con dificultad de mantenimiento.

---

## Criterios

| # | Criterio | Cómo medir | Umbral |
|---|---|---|---|
| 3.1 | LOC totales por módulo | `find ... -name "*.cs" -exec wc -l {} +` | Balance entre módulos ±2× |
| 3.2 | LOC por archivo | Top N archivos | Verde ≤300 / Rojo >500 |
| 3.3 | Método/función LOC | Análisis estático (SonarQube, NDepend) | ≤100 / >200 |
| 3.4 | Ciclomática complexity | Cyclomatic complexity por método | ≤10 / >20 |
| 3.5 | Cognitive complexity | SonarQube cognitive | ≤15 / >25 |
| 3.6 | Profundidad anidamiento | `if`/`for` nesting | ≤4 / >6 |
| 3.7 | Single Responsibility violado | # responsabilidades por clase | ≤5 / >10 |
| 3.8 | God classes/entities | LOC + count fields + count methods | ≤200 LOC / >400 |
| 3.9 | Duplicación de código | jscpd, dotnet-format, sonar duplications | ≤3% / >10% |
| 3.10 | Acoplamiento aferente/eferente | NDepend / dependency-cruiser metrics | Ratio sano |

---

## Comandos de referencia

```bash
# LOC totales por módulo
for m in <Module1> <Module2>; do
  echo "=== $m ===";
  find src/Modules/$m -name "*.cs" -not -path "*Tests*" -not -path "*Migrations*" \
    -exec wc -l {} \; | awk '{sum+=$1} END {print sum " LOC"}';
done

# Top 15 archivos por LOC
find src -name "*.cs" -not -path "*Tests*" -not -path "*Migrations*" \
  -exec wc -l {} \; | sort -rn | head -15

# Archivos > umbral
find src -name "*.cs" -exec awk 'END{print FILENAME, NR}' {} \; | awk '$2>500'

# Duplicación (TS)
npx jscpd src/ --threshold 3

# Duplicación (.NET via SonarQube CLI)
dotnet sonarscanner begin /k:"project" ...
dotnet build
dotnet sonarscanner end ...
```

---

## Evidencia esperada

- Tabla LOC por módulo
- Lista top 20 archivos por LOC con archivo + ruta
- Conteo de archivos > 300 LOC y > 500 LOC
- Reporte SonarQube/NDepend (si disponible) con ciclomática
- % de duplicación reportado

---

## Umbrales detallados

| Categoría | Verde | Ámbar | Rojo |
|---|---|---|---|
| Archivo `.cs`/`.ts` | ≤300 | 300-500 | >500 |
| Método LOC | ≤50 | 50-100 | >100 |
| Ciclomática método | ≤10 | 10-15 | >15 |
| Cognitive complexity | ≤15 | 15-25 | >25 |
| Duplicación global | ≤3% | 3-7% | >7% |
| Nesting profundidad | ≤3 | 4-5 | >5 |
| Public methods por clase | ≤10 | 10-20 | >20 |
| Fields por entidad | ≤15 | 15-25 | >25 |

---

## Anti-patrones

- Handler/Service > 200 LOC (god service)
- Entity con 20+ campos opcionales (mezcla bounded contexts)
- Método con 5+ niveles de `if` anidados
- Función que combina parsing + validación + persistencia + audit
- Clase con 30+ métodos públicos
- 10%+ duplicación entre módulos

---

## Refactor sugerido

| Síntoma | Refactor |
|---|---|
| God handler 500+ LOC | Extract Service objects (`<Action>Service`) en Domain |
| God entity con 2 ciclos de vida | Split entity o agregar discriminador TPH |
| Métodos largos | Extract Method por bloques semánticos |
| Anidamiento profundo | Guard clauses + early return |
| Duplicación cross-module | Mover a Shared/Kernel |

---

## Referencias cruzadas

- Patrón CQRS handlers → [§02 Arquitectura](02-arquitectura.md)
- Cobertura de handlers grandes → [§07 Testing](07-testing-cobertura.md)
- Hot files (más modificados) → [§17 Deuda](17-deuda-tecnica.md)
