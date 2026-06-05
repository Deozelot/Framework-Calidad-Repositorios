# §16 — Backend (.NET/Node/Java)

[← Volver al índice](INDEX.md) · Anterior: [§15](15-frontend.md) · Siguiente: [§17](17-deuda-tecnica.md)

---

## Objetivo

Criterios específicos para apps backend: API design, error handling, CQRS, validators, idempotencia.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 16.1 | API versioning convention | Audit rutas `api/v1/`, headers |
| 16.2 | Response envelope consistente (`ApiResponse<T>`) | Audit controllers |
| 16.3 | Error handling centralizado (middleware) | `GlobalExceptionMiddleware` |
| 16.4 | FluentValidation / DataAnnotations | Conteo validators |
| 16.5 | DTOs separados de Domain entities | Audit |
| 16.6 | Repository pattern + UnitOfWork | Audit interfaces |
| 16.7 | CQRS si declarado | Conteo handlers |
| 16.8 | Domain events | Audit `*Event.cs` |
| 16.9 | TimeProvider abstraído (no `DateTime.Now`) | `grep` directo |
| 16.10 | Background jobs (Hangfire, IHostedService) | Audit |
| 16.11 | Idempotencia en endpoints POST críticos | Audit `IdempotencyKey` |

---

## Comandos de referencia

```bash
# Rutas API versioning
grep -rn "\[Route" src --include="*.cs" | grep -v Tests

# Routes sin v1
grep -rn "\[Route" src --include="*.cs" | grep -v "api/v" | grep -v Tests

# ApiResponse usage
grep -rn "ApiResponse<\|ApiResponse\." src --include="*.cs" | wc -l

# Controllers que retornan tipos crudos (no ApiResponse)
grep -rn "return Ok(" src --include="*.cs" | grep -v "ApiResponse"

# GlobalExceptionMiddleware
find . -name "GlobalExceptionMiddleware.cs" -o -name "*ExceptionFilter.cs"

# Validators
find src -name "*Validator.cs" | wc -l
find src -name "*Command.cs" | wc -l   # ratio cmd/validator should be ~1:1

# DTOs vs Entities
find src -name "*Dto.cs" -o -name "*Request.cs" -o -name "*Response.cs" | wc -l

# CQRS handlers
find src -name "*CommandHandler.cs" -not -path "*Tests*" | wc -l
find src -name "*QueryHandler.cs" -not -path "*Tests*" | wc -l

# Domain events
find src -name "*Event.cs" -path "*Domain*" | wc -l

# TimeProvider abstraction
grep -rn "DateTime\.Now\|DateTime\.UtcNow" src --include="*.cs" \
  | grep -v "Tests\|Migrations" | wc -l
grep -rn "TimeProvider\|IClock\|IDateTimeProvider" src --include="*.cs"

# Background jobs
grep -rn "IHostedService\|BackgroundService\|PeriodicTimer" src --include="*.cs"

# Idempotency
grep -rn "IdempotencyKey\|Idempotency-Key" src --include="*.cs"
```

---

## Evidencia esperada

- Tabla controllers + rutas + ¿usa v1?
- Conteo endpoints con/sin `ApiResponse<T>`
- Lista validators vs commands sin validator
- Conteo handlers CQRS por módulo
- Lista `DateTime.UtcNow` directo (rojo si > 0 en código de negocio)
- Lista background jobs registrados
- Endpoints idempotentes documentados

---

## Umbrales

| Métrica | Verde | Rojo |
|---|---|---|
| Rutas con `/v1/` | 100% | <50% |
| Endpoints con ApiResponse | 100% | <90% |
| Commands con Validator | 100% | <80% |
| `DateTime.UtcNow` en handlers | 0 (vía TimeProvider) | >5 |
| GlobalExceptionMiddleware | Sí | No |
| Try/catch en controllers | 0 (delegado a middleware) | >5 |
| Endpoints POST idempotentes (críticos) | 100% | <80% |

---

## API design — REST mínimo

```
GET    /api/v1/wells              200 OK   ApiResponse<PagedResult<WellDto>>
GET    /api/v1/wells/{id:guid}    200 OK   ApiResponse<WellDetailDto>
                                  404 NotFound
POST   /api/v1/wells              201 Created (Location header)
                                  400 BadRequest (validation)
                                  409 Conflict (business rule)
PUT    /api/v1/wells/{id:guid}    200 OK / 204 NoContent
                                  404 NotFound
                                  409 Conflict
DELETE /api/v1/wells/{id:guid}    204 NoContent
                                  404 NotFound
```

### Response envelope

```csharp
public sealed record ApiResponse<T>
{
    public bool Success { get; init; }
    public T? Data { get; init; }
    public ApiError? Error { get; init; }

    public static ApiResponse<T> Ok(T data) => new() { Success = true, Data = data };
    public static ApiResponse<T> Fail(string code, string? detail = null) =>
        new() { Success = false, Error = new(code, detail) };
}
```

---

## CQRS patterns

```csharp
// Command (escritura, retorna mínimo o void)
public sealed record CreateWellCommand(
    string Name,
    Guid OperatorId,
    string ContractId
) : ICommand<Result<WellCreatedDto>>;

internal sealed class CreateWellCommandHandler
    : ICommandHandler<CreateWellCommand, Result<WellCreatedDto>>
{
    public async Task<Result<WellCreatedDto>> HandleAsync(
        CreateWellCommand cmd, CancellationToken ct)
    {
        // 1. Validate
        // 2. Business rules
        // 3. Persist
        // 4. Audit
        // 5. Return DTO
    }
}

// Query (lectura, projection directa)
public sealed record GetWellByIdQuery(Guid WellId) : IQuery<WellDetailDto?>;

internal sealed class GetWellByIdQueryHandler
    : IQueryHandler<GetWellByIdQuery, WellDetailDto?>
{
    public async Task<WellDetailDto?> HandleAsync(GetWellByIdQuery q, CancellationToken ct)
    {
        return await _db.Wells
            .AsNoTracking()
            .Where(w => w.Id == q.WellId)
            .Select(w => new WellDetailDto { ... })
            .FirstOrDefaultAsync(ct);
    }
}
```

---

## Anti-patrones

- Controllers con lógica de negocio (acceso DbContext directo)
- Endpoints retornan entities en lugar de DTOs (leak Domain)
- Try/catch en controller → debe ser middleware
- `Roles=` hardcoded en `[Authorize]` (usar policies)
- Sin validation = inputs llegan crudos al handler
- Commands lanzan excepciones para errores de negocio (usar `Result<T>`)
- Handler retorna entity completa cuando UI necesita 3 campos
- Synchronous I/O (`File.Read*` sin async)
- `DateTime.Now`/`UtcNow` directo (no testeable)
- Background jobs sin retry policy / sin error handling
- POST no idempotente con cliente con retry → duplicados
- Endpoints sin paginación retornan colecciones ilimitadas
- Sin rate limiting → abuse trivial
- HTTP clients `new HttpClient()` (socket exhaustion)
- Single DbContext compartido entre módulos (acoplamiento)

---

## Validators — FluentValidation

```csharp
public sealed class CreateOperatorCommandValidator : AbstractValidator<CreateOperatorCommand>
{
    public CreateOperatorCommandValidator()
    {
        RuleFor(c => c.Code)
            .NotEmpty()
            .MaximumLength(50)
            .Matches("^[A-Z0-9_-]+$")
            .WithMessage("Code must be uppercase alphanumeric");

        RuleFor(c => c.Name)
            .NotEmpty()
            .MaximumLength(200);

        RuleFor(c => c.TaxId)
            .Matches(@"^\d{6,10}(-\d)?$")
            .When(c => !string.IsNullOrEmpty(c.TaxId));
    }
}
```

---

## TimeProvider (.NET 8+)

```csharp
// Mal
DateTime.UtcNow

// Bien
public class MyService(TimeProvider clock) {
    var now = clock.GetUtcNow();
}

// Tests con FakeTimeProvider
var fake = new FakeTimeProvider(DateTimeOffset.Parse("2026-01-15"));
var svc = new MyService(fake);
fake.Advance(TimeSpan.FromDays(7));
```

---

## Background jobs patterns

```csharp
public class Forma101CaducidadJob : BackgroundService
{
    private readonly IServiceProvider _sp;
    private readonly TimeProvider _clock;

    protected override async Task ExecuteAsync(CancellationToken ct)
    {
        using var timer = new PeriodicTimer(TimeSpan.FromHours(1), _clock);
        while (await timer.WaitForNextTickAsync(ct))
        {
            using var scope = _sp.CreateScope();
            var dispatcher = scope.ServiceProvider.GetRequiredService<ICommandDispatcher>();
            await dispatcher.DispatchAsync(new ExpireForma101DraftsCommand(), ct);
        }
    }
}
```

---

## Idempotencia POST

```csharp
[HttpPost]
public async Task<IActionResult> Create(
    [FromHeader(Name = "Idempotency-Key")] string idempotencyKey,
    [FromBody] CreateRequest req,
    CancellationToken ct)
{
    var existing = await _idempotencyStore.GetAsync(idempotencyKey, ct);
    if (existing is not null) return Ok(existing);

    var result = await _dispatcher.DispatchAsync(new CreateCommand(req), ct);
    await _idempotencyStore.SaveAsync(idempotencyKey, result, ct);
    return CreatedAtAction(...);
}
```

---

## Referencias cruzadas

- Tests deterministas → [§07 Testing](07-testing-cobertura.md)
- Multitenancy enforcement → [§14 Datos](14-datos-persistencia.md)
- Performance handlers grandes → [§03 Mantenibilidad](03-mantenibilidad.md)
- OpenAPI documentation → [§09 Documentación](09-documentacion.md)
