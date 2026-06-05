# §12 — Performance y Escalabilidad

[← Volver al índice](INDEX.md) · Anterior: [§11](11-observabilidad.md) · Siguiente: [§13](13-dependencias.md)

---

## Objetivo

Evaluar comportamiento bajo carga, eficiencia de queries, footprint del bundle, latencias.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 12.1 | Paginación server-side en listados | Audit `pageSize` hardcoded vs param |
| 12.2 | Queries N+1 detectadas | EF Core sensitive logs / SonarQube |
| 12.3 | Índices en columnas de filtro/join | Migrations review |
| 12.4 | Caching strategy (memoria, Redis, CDN) | Conteo `IMemoryCache`/`IDistributedCache` |
| 12.5 | Async/await universal en I/O | Audit |
| 12.6 | Streaming para responses grandes | `IAsyncEnumerable` en queries |
| 12.7 | Bundle size frontend | `webpack-bundle-analyzer`, Angular budgets |
| 12.8 | Lazy loading routes | Audit `loadChildren` Angular |
| 12.9 | Tree shaking efectivo | Bundle analyzer |
| 12.10 | Load testing baseline | k6, JMeter results |
| 12.11 | P95/P99 latencias documentadas | SLO doc + dashboards |

---

## Comandos de referencia

```bash
# pageSize hardcoded en frontend
grep -rn "pageSize:\s*[0-9]\+\|pageSize=[0-9]\+" src --include="*.ts" \
  | grep -v "pageSize:\s*20\|pageSize:\s*50"

# .Result/.Wait() bloqueantes
grep -rn "\.Result\b\|\.Wait()" src --include="*.cs" | grep -v "Tests"

# AsNoTracking en queries de listado
grep -rn "AsNoTracking" src --include="*.cs" | wc -l
grep -rln "DbSet<.*>\|IQueryable<" src/Modules/*/Application --include="*.cs"

# Bundle Angular
ng build --configuration=production --stats-json
npx webpack-bundle-analyzer dist/<app>/stats.json

# Bundle Vite
npx vite build --report

# Lazy routes Angular
grep -rn "loadChildren\|loadComponent" src --include="*.ts" | wc -l

# Load test k6
k6 run --vus 50 --duration 2m loadtest.js

# DB indexes
grep -rn "HasIndex\|IsClustered\|builder\.Property.*HasColumnType" \
  src --include="*Configuration.cs"
```

---

## Evidencia esperada

- Lista endpoints con paginación server-side
- Lista endpoints sin paginación (devuelven all)
- Reporte bundle Angular con tamaño por chunk
- Resultado k6/JMeter con throughput, latency P95/P99
- Lista índices DB con columnas
- Cache hit rate (si Redis/MemoryCache instrumentado)

---

## Umbrales

| Métrica | Verde | Rojo |
|---|---|---|
| Bundle main inicial | <500KB gzip | >2MB |
| Lazy chunks por route | <300KB | >1MB |
| P95 GET endpoints | <500ms | >2000ms |
| P95 POST endpoints | <1500ms | >5000ms |
| Throughput (req/s) baseline | >100 | <20 |
| Cache hit rate | >80% | <40% |
| N+1 queries detectadas | 0 | ≥1 |
| `pageSize: 1000` hardcoded | 0 | ≥1 |

---

## Anti-patrones

- `getAll()` sin paginación
- `pageSize: 1000` para mostrar 20 elementos
- LINQ con `.ToList()` antes de `.Where()` (materialización innecesaria)
- N+1 queries por falta de `Include`
- `var entities = await _db.Set<T>().ToListAsync()` luego filtrar en memoria
- Bundle frontend > 5MB
- Imágenes sin optimizar (PNG 4MB → debería WebP/AVIF)
- Sin lazy loading routes (todo en main bundle)
- Frontend pide BD entera para autocomplete
- Sin caching de catálogos read-mostly
- API que retorna entity con 50 campos cuando UI usa 3

---

## EF Core performance checklist

```csharp
// Bien: AsNoTracking + Select projection
var dtos = await _db.Wells
    .AsNoTracking()
    .Where(w => w.OperatorId == operatorId)
    .Select(w => new WellSummaryDto
    {
        Id = w.Id,
        Name = w.Name,
        State = w.State
    })
    .OrderBy(w => w.Name)
    .Skip((page - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync(ct);

// Mal: materializa entidad completa
var wells = await _db.Wells
    .Where(w => w.OperatorId == operatorId)
    .ToListAsync();
return wells.Select(w => new WellSummaryDto { ... }).Skip(0).Take(20);

// Mal N+1:
foreach (var well in wells)
    var formas = await _db.Formas.Where(f => f.WellId == well.Id).ToListAsync();

// Bien:
var wellsWithFormas = await _db.Wells
    .Include(w => w.Formas)
    .Where(...).ToListAsync();
```

---

## Frontend performance checklist

- Lazy load por route (`loadChildren`)
- OnPush change detection (Angular)
- Signals para granular reactivity
- Virtual scroll para listas >100 items
- Debounce inputs de búsqueda (300ms)
- Imágenes responsive con `<picture>`
- WebP/AVIF + fallback
- Code splitting por bounded context
- Preconnect a APIs externas (`<link rel="preconnect">`)
- Service Worker para cache offline (PWA)
- `defer`/`async` en scripts no críticos
- HTTP/2 push o preload de fonts críticas

---

## Load testing baseline (k6)

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 20 },
    { duration: '1m', target: 50 },
    { duration: '30s', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],
    http_req_failed: ['rate<0.01'],
  },
};

export default function () {
  const res = http.get('https://api.example.com/wells?page=1&pageSize=20');
  check(res, { 'status 200': r => r.status === 200 });
  sleep(1);
}
```

---

## Referencias cruzadas

- Métricas P95 → [§11 Observabilidad](11-observabilidad.md)
- Índices migraciones → [§14 Datos](14-datos-persistencia.md)
- Bundle Angular → [§15 Frontend](15-frontend.md)
- Paginación API → [§16 Backend](16-backend.md)
