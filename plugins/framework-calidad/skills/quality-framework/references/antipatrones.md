# §21 — Anti-patrones: Señales de Alerta

[← Volver al índice](INDEX.md) · Anterior: [§20](20-plantilla-informe.md) · Siguiente: [§22](22-score-global.md)

---

## Objetivo

Catálogo de "olores" inmediatos que justifican análisis profundo. Si encuentras uno, el repo necesita atención.

---

## Olores estructurales (§01)

- `publish/`, `dist/`, `node_modules/`, `bin/`, `obj/` committeados
- `.env`, `.env.local`, `appsettings.Local.json` versionados
- Múltiples convenciones de naming coexistiendo (`HU_GOP_06_*` + `Forma_204_*`)
- IDs colisionados (HU-010 dos veces en `Trazabilidad/`)
- Carpetas vacías o "placeholder" sin contenido real
- 25+ dirs en raíz sin agrupación
- README ausente o "Hello World" sin actualizar
- Lockfile sin manifiesto hermano (`package-lock.json` sin `package.json`)
- Dos carpetas equivalentes con propósito solapado (`scripts/` raíz + `backend/scripts/`)

---

## Olores arquitectónicos (§02)

- Module A importa internals de Module B directamente
- Domain importa EF Core / ASP.NET / HttpClient
- Application importa Infrastructure
- Controller con `_db.Wells.Where(...)` (acceso DbContext directo)
- ADRs ausentes pero decisiones arquitectónicas tomadas
- Patrón declarado (CQRS) pero handlers mezclan repositorios con DbContext
- Bounded contexts mezclados sin frontera explícita
- Shared kernel hipertrofiado con "utilidades" no compartidas realmente
- DbContext único para todos los módulos
- 1 controller monolítico con CRUD + review + signing + locking (10+ endpoints)

---

## Olores de tamaño (§03)

- Archivos > 1000 LOC
- Handler/Service > 200 LOC
- Entity con 20+ campos opcionales (mezcla bounded contexts)
- Método con 5+ niveles de `if` anidados
- Función que combina parsing + validación + persistencia + audit
- Clase con 30+ métodos públicos
- 10%+ duplicación entre módulos
- Controller > 400 LOC
- Test file > 800 LOC (split por concern)
- DbContext > 200 LOC con 50+ DbSet

---

## Olores de código (§05)

- Comentarios `// TODO: fix this` sin issue link
- `catch (Exception) { }` vacíos
- Tests con `[Skip]` o `xit` mergeados a main
- `// HACK:`, `// XXX:` sin contexto
- Imports `* as X` masivos
- Funciones `function doStuff()` o `Handle()` sin nombre semántico
- `any`/`object`/`dynamic` proliferando
- `Console.WriteLine` / `console.log` en producción
- `.Result` / `.Wait()` blocking calls
- Magic numbers `if (count > 86400)` sin constante
- `var temp = ...` `var data = ...` sin contexto
- Lambda con 50 LOC inline
- Métodos privados que solo se llaman desde un sitio (inline)
- Comentarios que duplican lo obvio (`// increment i`)

---

## Olores de seguridad (§06)

- `appsettings.json` con `"ConnectionString": "Server=...;Password=..."`
- API key hardcoded: `const API_KEY = "sk-..."`
- `.env` con secrets en commit history
- Endpoint público sin `[Authorize]` que retorna datos sensibles
- `[Authorize(Roles=...)]` en lugar de policy-based authz
- Concatenación SQL: `$"SELECT * FROM users WHERE name='{name}'"`
- Angular `[innerHTML]="userInput"` sin sanitizar
- BCrypt con `WorkFactor < 12`
- MD5/SHA1 para passwords
- TLS 1.0/1.1 habilitados
- CORS `AllowAnyOrigin()` en producción
- Errores que exponen stack trace al cliente
- JWT sin expiración corta (>24h sin refresh)
- Logs que incluyen passwords/tokens en plain text

---

## Olores de tests (§07)

- `[Skip]`, `xit`, `xdescribe` mergeados a main
- `DateTime.UtcNow` directo en assertion
- Tests dependientes de orden de ejecución
- `Thread.Sleep(5000)` para sincronizar
- Snapshot tests sin revisión (todo se acepta)
- E2E que toca BD real y deja datos basura
- Test "happy path" único sin edge cases
- `Assert.True(true)` (test que no asserta nada)
- Test que pasa porque catch silencia error
- Suite total > 30 min en CI sin paralelización
- Cobertura "alta" por archivos auto-generados (migrations, DTOs)
- Sin tests para reglas de negocio críticas
- Mocks que devuelven "always success" sin reflect comportamiento real

---

## Olores de CI/CD (§08)

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
- Pipeline tiempo > 1h sin justificación
- Multiple jobs serializados que podrían paralelizar
- "Pipeline rojo aceptable, mergeamos igual"

---

## Olores de documentación (§09)

- README "Project X — TODO: write docs"
- ADRs como issues de GitHub (se pierden con tiempo)
- OpenAPI con `description: "TODO"`
- Quickstart que asume entorno preconfigurado del autor
- Doc dice "ejecutar `make build`" pero `Makefile` no existe
- Diagramas `.png` no editables vs `.puml`/`.mmd` editables
- Storybook con build roto en main
- CHANGELOG auto-generado sin curaduría
- `.env.example` con secrets reales
- Comentarios en código que duplican lo obvio
- Doc actualizada hace 6 meses cuando código cambia diariamente
- README en inglés pero código y comentarios en español (incoherente)

---

## Olores de procesos (§10)

- Commits "fix stuff", "WIP", "asdf"
- PRs gigantes (5000 LOC, 50 archivos)
- PRs sin descripción ni issue link
- Branch viva > 30 días (merge hell garantizado)
- Release sin changelog
- Hotfix directo a main sin proceso
- Tags no semver (`release-2024`, `prod-v2-final-FINAL`)
- DoD documentado pero no aplicado
- Trazabilidad solo cuando alguien pregunta
- Force push a main (catástrofe)
- Múltiples PRs editando mismo archivo simultáneamente
- Approvers que aprueban sin leer

---

## Olores de observabilidad (§11)

- `Console.WriteLine` para logging en producción
- Logs sin correlation ID — imposible trazar flujo
- Logs `LogInformation` para todo (sin niveles)
- Logs con string concatenation `$"User {id}"` (no estructurado)
- Sin health endpoints — imposible para Kubernetes/load balancer
- Sin alerting — descubres incidentes por reporte de usuario
- Dashboards "vanity metrics" (login count) sin business KPIs
- Error tracking deshabilitado en producción "por privacidad"
- Métricas custom sin documentar
- Retención de logs <7 días

---

## Olores de performance (§12)

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

## Olores de dependencias (§13)

- `"react": "*"` o `"latest"` — version drift garantizado
- Sin lockfile committeado — builds no reproducibles
- `npm install --force` para callar warnings
- Dependencias prod en `devDependencies` o viceversa
- Frontend importa `moment.js` (deprecated)
- Lodash full cuando solo se usan 2 funciones
- Paquetes con last publish > 2 años (abandono)
- Single-maintainer packages críticos (bus factor)
- Sin Dependabot → vulnerabilidades acumulan

---

## Olores de datos (§14)

- Migración `Down()` lanza `NotImplementedException`
- Migración re-fechada manualmente (timestamps no monotónicos)
- FK sin índice = full table scan en join
- Soft delete inconsistente (`IsActive`, `DeletedAt`, `IsDeleted` mezclados)
- Audit columns nullable que deberían ser NOT NULL
- DbContext único compartido por todos los módulos
- Sin global query filter = riesgo de leak cross-tenant
- Backup automatizado sin test de Restore regular
- BLOB grandes en mismas tablas que datos transaccionales
- Migración que cambia tipo de columna sin shadow column

---

## Olores de backend (§16)

- Controllers con lógica de negocio
- Endpoints retornan entities en lugar de DTOs
- Try/catch en controller (debe ser middleware)
- `Roles=` hardcoded en `[Authorize]`
- Commands lanzan excepciones para errores de negocio (usar `Result<T>`)
- Handler retorna entity completa cuando UI necesita 3 campos
- `DateTime.Now`/`UtcNow` directo (no testeable)
- POST no idempotente con cliente que retry → duplicados
- Endpoints sin paginación
- HTTP clients `new HttpClient()` (socket exhaustion)
- Visibilidad inconsistente (mitad `internal sealed`, mitad `public sealed`)

---

## Olores de frontend (§15)

- ChangeDetection default + Observable pipe (re-renders innecesarios)
- `console.log` en producción
- `subscribe()` sin `unsubscribe()`
- Lógica de negocio en componente vs en service/store
- Routing sin `loadChildren` (todo en bundle main)
- Sin `OnPush` en componentes con muchos props
- Bindings inseguros `[innerHTML]` sin sanitizar
- A11y ignorado: sin labels, contraste insuficiente
- i18n hardcodeado en lugar de claves
- Imports `* as X` (rompen tree shaking)
- Mezcla Signals + RxJS sin razón clara
- `any` proliferando en lugar de tipos

---

## Olores de deuda (§17)

- Deuda invisible: no hay `TECH_DEBT.md` ni label
- "Lo arreglamos después" sin issue creada
- TODOs con > 6 meses sin atender
- Cobertura decreciente sprint tras sprint
- Build time creciendo sin contención
- Tests flaky tolerados ("solo correrlo de nuevo")
- Bus factor 1 en módulo crítico
- Hot file que cambia 50 veces/mes sin refactor
- Stale branches abandonadas
- Issues > 1 año abiertas sin acción
- "Big rewrite" planeado en lugar de refactor incremental

---

## Cómo priorizar al detectar olores

Por severidad:

1. **🚨 CRÍTICO inmediato** — atender hoy
   - Secrets en código actual / historia
   - Vulnerabilidades CRITICAL sin patch
   - Endpoint sensible sin AuthZ
   - Force push permitido a main
   - Backups sin test de restore

2. **🟠 ALTO — sprint actual**
   - Vulnerabilidades HIGH
   - Tests skipped en main
   - Build/CI rojo persistente
   - Sin observabilidad básica
   - Handler 600+ LOC

3. **🟡 MEDIO — próximo sprint**
   - Documentación obsoleta
   - Bundle > 3MB
   - Cobertura <60%
   - Deuda con > 3 meses

4. **🟢 BAJO — backlog**
   - Refactors cosmeticos
   - Naming inconsistente
   - TODOs sin contexto

---

## Referencias cruzadas

Cada anti-patrón referenciado tiene su sección con detalle completo (§01 a §17).
