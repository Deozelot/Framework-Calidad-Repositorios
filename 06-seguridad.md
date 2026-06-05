# §06 — Seguridad

[← Volver al índice](INDEX.md) · Anterior: [§05](05-buenas-practicas-codigo.md) · Siguiente: [§07](07-testing-cobertura.md)

---

## Objetivo

Auditar secretos, vulnerabilidades de dependencias, AuthZ, OWASP Top 10, hardening de aplicación.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 6.1 | Secrets en código | `gitleaks`, `truffleHog`, `git-secrets` |
| 6.2 | Secrets en historia git | `gitleaks detect --log-opts="--all"` |
| 6.3 | Dependencias vulnerables | `npm audit`, `dotnet list package --vulnerable`, Dependabot |
| 6.4 | CVE alerts pendientes | GitHub Security tab |
| 6.5 | Auth/AuthZ presente en endpoints | Audit `[Authorize]` por controller action |
| 6.6 | Rate limiting | Middleware + policies declaradas |
| 6.7 | CORS configurado restrictivo | Inspección `program.cs`/`main.ts` |
| 6.8 | HTTPS/TLS enforced | Config |
| 6.9 | SQL injection — parametrización | `grep "SqlCommand\|FromSqlRaw"` sin parámetros |
| 6.10 | XSS — sanitización inputs | Audit binding inseguro (`[innerHTML]` en Angular) |
| 6.11 | CSRF protection | Token validation middleware |
| 6.12 | Headers de seguridad | `helmet` (Node), `AddGopSecurityHardening` o equivalente |
| 6.13 | Audit log de operaciones sensibles | Conteo `_audit.LogAsync` en handlers |
| 6.14 | Data classification (PII, PCI, etc.) | `[DataClassification]` attribute audit |
| 6.15 | OWASP Top 10 review periódico | Última auditoría documentada |
| 6.16 | SSRF protection en HTTP clients | `SsrfProtectionHandler` en `IHttpClientFactory` |
| 6.17 | Password storage (BCrypt/Argon2, no MD5/SHA1) | Inspección hashing |

---

## Comandos de referencia

```bash
# Secrets en working tree
gitleaks detect --source . --report-format json --report-path gitleaks-report.json

# Secrets en historia completa
gitleaks detect --source . --log-opts="--all" --report-path gitleaks-history.json

# Vulnerable deps (.NET)
dotnet list package --vulnerable --include-transitive

# Vulnerable deps (Node)
npm audit --json
npm audit --audit-level=high

# Container scan
trivy image <image-name>:<tag>

# OWASP ZAP scan (DAST)
docker run -t owasp/zap2docker-stable zap-baseline.py -t https://<url>

# Endpoints sin [Authorize]
grep -B2 "\[HttpGet\|\[HttpPost\|\[HttpPut\|\[HttpDelete" src --include="*.cs" -r \
  | grep -v "\[Authorize\]\|\[AllowAnonymous\]"

# Auditoría operaciones
grep -rn "_audit\.LogAsync\|AuditLog\.Create" src --include="*.cs" | wc -l

# SQL injection riesgo
grep -rn "FromSqlRaw\|ExecuteSqlRaw\|new SqlCommand" src --include="*.cs"

# XSS riesgo Angular
grep -rn "innerHTML\|bypassSecurityTrust" src --include="*.ts"

# Hashing débil
grep -rn "MD5\|SHA1\|MD5\.Create\|SHA1Managed" src --include="*.cs"
```

---

## Evidencia esperada

- Reporte gitleaks (HTML/JSON) — 0 findings ideal
- Output `dotnet list --vulnerable` (vacío ideal)
- Output `npm audit --audit-level=high` (0 high/critical)
- Lista endpoints sin `[Authorize]` (excepto login/health)
- Reporte OWASP ZAP baseline
- Conteo audit log calls por handler de escritura
- Última fecha de auditoría OWASP Top 10

---

## Umbrales

| Métrica | Verde | Rojo |
|---|---|---|
| Secrets en código actual | 0 | ≥1 |
| Secrets en historia | 0 | ≥1 (revocar + rotate) |
| Vulnerabilidades CRITICAL | 0 | ≥1 |
| Vulnerabilidades HIGH | 0 | ≥3 |
| Endpoints sin AuthZ | 0 (excluido health/login) | ≥1 |
| Hashing MD5/SHA1 para passwords | 0 | ≥1 |

---

## Anti-patrones críticos

- `appsettings.json` con `"ConnectionString": "Server=...;Password=..."`
- API key en variable `const API_KEY = "sk-..."`
- `.env` o `appsettings.Local.json` en commit history
- Endpoint público sin `[Authorize]` que retorna datos sensibles
- `[Authorize(Roles=...)]` en lugar de policy-based authz
- Concatenación SQL: `$"SELECT * FROM users WHERE name='{name}'"`
- Angular `[innerHTML]="userInput"` sin sanitizar
- BCrypt con `WorkFactor < 12`
- TLS 1.0/1.1 habilitados
- CORS `AllowAnyOrigin()` en producción

---

## Auditoría regulatoria recomendada

Para sistemas con datos regulados (ANH, salud, financiero):

- **OWASP ASVS Level 2** mínimo
- Audit log inmutable con retención según regulación
- Cifrado en reposo (TDE SQL Server, Azure Disk Encryption)
- Cifrado en tránsito (TLS 1.2+ exclusivo)
- Trazabilidad completa de acceso a datos sensibles
- Separación de duties (no admin = auditor)
- Penetration test anual por tercero

---

## Referencias cruzadas

- Dependencias vulnerables → [§13 Dependencias](13-dependencias.md)
- AuthZ patterns → [§16 Backend](16-backend.md)
- Headers seguridad frontend → [§15 Frontend](15-frontend.md)
- Encryption at rest → [§14 Datos](14-datos-persistencia.md)
