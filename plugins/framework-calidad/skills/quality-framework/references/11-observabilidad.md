# §11 — Observabilidad y Operación

[← Volver al índice](INDEX.md) · Anterior: [§10](10-procesos-trazabilidad.md) · Siguiente: [§12](12-performance.md)

---

## Objetivo

Evaluar capacidad de detectar, diagnosticar y resolver problemas en producción.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 11.1 | Structured logging (Serilog, Pino) | Conteo `_logger.LogInformation` con scopes |
| 11.2 | Log aggregation centralizado (Seq, ELK, Datadog) | Config |
| 11.3 | Correlation IDs en requests | Middleware presence |
| 11.4 | Métricas (OpenTelemetry, Prometheus) | Endpoint `/metrics` |
| 11.5 | Tracing distribuido | OpenTelemetry exporters |
| 11.6 | Health checks | `/health`, `/ready`, `/live` |
| 11.7 | Alerting configurado | PagerDuty/Opsgenie + rules |
| 11.8 | SLOs/SLIs definidos | `docs/slo.md` |
| 11.9 | Dashboards operacionales | Grafana/Datadog screenshots |
| 11.10 | Error tracking (Sentry, Rollbar) | DSN configurado + reports |

---

## Comandos de referencia

```bash
# Logging usage
grep -rn "ILogger<\|_logger\." src --include="*.cs" | wc -l
grep -rn "LogInformation\|LogWarning\|LogError" src --include="*.cs" \
  | awk -F: '{print $3}' | sort | uniq -c | sort -rn

# Correlation ID middleware
grep -rn "CorrelationId\|TraceId\|Activity.Current" src --include="*.cs"

# Health checks
grep -rn "AddHealthChecks\|MapHealthChecks\|IHealthCheck" src --include="*.cs"

# OpenTelemetry
grep -rn "AddOpenTelemetry\|TraceSource\|ActivitySource" src --include="*.cs"

# Endpoint /health, /metrics test
curl -sf http://localhost:5000/health
curl -sf http://localhost:5000/metrics | head
```

---

## Evidencia esperada

- Lista de log sinks configurados (Console, File, Seq, ELK)
- Existencia + endpoint health/ready/live
- Lista métricas custom expuestas
- Screenshot dashboards principales
- Lista de alertas configuradas con threshold
- SLO documento con objetivo + medición

---

## Stack observabilidad recomendado

| Capa | .NET | Node | Universal |
|---|---|---|---|
| Logs | Serilog + Seq | Pino + Loki | OpenTelemetry Logs + Loki/ELK |
| Métricas | OpenTelemetry .NET + Prometheus | prom-client | Prometheus + Grafana |
| Tracing | OpenTelemetry .NET + Jaeger | OpenTelemetry JS + Jaeger | Tempo, Honeycomb, Datadog |
| APM | Application Insights, Datadog | New Relic, Datadog | Datadog, Dynatrace |
| Errors | Sentry .NET | Sentry JS | Sentry |
| Alerting | PagerDuty + Grafana Alerts | PagerDuty | Opsgenie, PagerDuty |
| Dashboards | Grafana | Grafana | Grafana, Datadog |

---

## Three pillars + logs structurados

```csharp
// Bien: log con scope + estructura
using (_logger.BeginScope(new { CorrelationId = ctx.TraceIdentifier, UserId = userId }))
{
    _logger.LogInformation("Forma101 {Forma101Id} aprobada por {ApproverId}",
        forma101Id, approverId);
}

// Mal: log plano
_logger.LogInformation($"Aprobada forma {forma101Id}");
```

---

## Health check niveles

| Endpoint | Propósito | Cuándo falla |
|---|---|---|
| `/health/live` | ¿El proceso responde? | Crash, deadlock |
| `/health/ready` | ¿Está listo para tráfico? | DB no responde, dep externa caída |
| `/health` | Composición de las anteriores | Cualquiera de las dos |

```csharp
builder.Services.AddHealthChecks()
    .AddDbContextCheck<AuthDbContext>("auth-db")
    .AddSqlServer(connectionString, name: "sqlserver")
    .AddCheck<ControlDocHealthCheck>("controldoc-integration")
    .AddCheck<SolarHealthCheck>("solar-integration");

app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = _ => false  // solo verifica el proceso
});
app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready")
});
```

---

## SLOs ejemplo (sistemas regulatorios)

| SLI | SLO | Window |
|---|---|---|
| Disponibilidad API | 99.5% | 30 días rolling |
| Latencia P95 GET endpoints | <500ms | 7 días |
| Latencia P95 POST endpoints | <1500ms | 7 días |
| Tasa de error 5xx | <0.5% | 30 días |
| Tiempo radicación forma (E2E) | <3s | 7 días |
| Disponibilidad integración SOLAR | 99% | 30 días |

---

## Anti-patrones

- `Console.WriteLine` para logging en producción
- Logs sin correlation ID — imposible trazar flujo
- Logs `LogInformation` para todo (sin niveles)
- Logs con string concatenation `$"User {id}"` (no estructurado, no indexable)
- Sin health endpoints — imposible para Kubernetes/load balancer
- Sin alerting — descubres incidentes por reporte de usuario
- Dashboards vanity metrics (login count) sin business KPIs
- Error tracking deshabilitado en producción "por privacidad"
- Métricas custom sin documentar
- Retención de logs <7 días (poco para investigar)

---

## Métricas mínimas (RED + USE)

**RED (servicios):**
- **R**ate — requests/segundo
- **E**rrors — % errores
- **D**uration — latencia P50/P95/P99

**USE (recursos):**
- **U**tilization — % uso CPU/memoria
- **S**aturation — cola pendiente
- **E**rrors — errores hardware/network

**Business metrics:**
- Formas radicadas/día
- Tiempo promedio aprobación
- Tasa rechazo por motivo

---

## Referencias cruzadas

- Audit log → [§06 Seguridad](06-seguridad.md)
- Latencia P95 → [§12 Performance](12-performance.md)
- Deploy notifications → [§08 CI/CD](08-cicd.md)
