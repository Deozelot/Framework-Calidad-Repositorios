# §14 — Calidad de Datos y Persistencia

[← Volver al índice](INDEX.md) · Anterior: [§13](13-dependencias.md) · Siguiente: [§15](15-frontend.md)

---

## Objetivo

Auditar capa de datos: migraciones, integridad referencial, índices, multitenancy, backup.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 14.1 | Migraciones reversibles (`Down()` válida) | Audit cada migration |
| 14.2 | Timestamps migraciones monotónicos | Lista cronológica vs orden archivo |
| 14.3 | Foreign keys + cascade rules explícitas | Configurations EF/Liquibase |
| 14.4 | Índices en FK y columnas filtro | Audit migrations |
| 14.5 | Constraints unique declaradas | Audit |
| 14.6 | Soft delete consistente | Convención (`IsActive`, `DeletedAt`) |
| 14.7 | Audit columns (CreatedAt/UpdatedAt/CreatedBy) | Convención + EF interceptor |
| 14.8 | Multitenancy enforcement (Global Query Filter) | DbContext audit |
| 14.9 | Backup strategy documentada | Runbook |
| 14.10 | Schemas por bounded context | DbContext separado por módulo |

---

## Comandos de referencia

```bash
# Migraciones .NET
find . -name "*.cs" -path "*Migrations*" -not -name "*.Designer.cs" \
  -not -path "*/bin/*" | sort

# Down() implementation check
grep -L "protected override void Down" \
  $(find . -name "*.cs" -path "*Migrations*" -not -name "*.Designer.cs")

# Timestamps monotónicos (detectar re-fechados)
find . -name "*.cs" -path "*Migrations*" -not -name "*.Designer.cs" \
  | sed 's|.*/\([0-9]*\)_.*|\1|' | sort -c

# FK + indexes en configurations
grep -rn "HasForeignKey\|HasIndex\|OnDelete" \
  src --include="*Configuration.cs"

# Global Query Filter
grep -rn "HasQueryFilter\|IgnoreQueryFilters" src --include="*.cs"

# Audit columns
grep -rn "CreatedAt\|UpdatedAt\|CreatedBy" src/Modules/*/Domain/Entities --include="*.cs" | wc -l

# Schemas DB
grep -rn "HasDefaultSchema\|ToTable.*schema" src --include="*.cs"
```

---

## Evidencia esperada

- Lista cronológica de migraciones (timestamps deben ser monotónicos)
- Lista migraciones sin `Down()` válida
- Diagrama ERD por bounded context
- Conteo FK declaradas vs referencias por convención
- Lista global query filters activos
- Política backup: frecuencia, retención, ubicación, restore test frequency

---

## Umbrales

| Métrica | Verde | Rojo |
|---|---|---|
| Migraciones sin Down() | 0 | ≥1 |
| Timestamps no monotónicos | 0 | ≥1 |
| FK sin índice | 0 | ≥1 |
| Tablas sin audit columns | 0 (excluyendo lookups) | ≥1 |
| DbContext compartido entre módulos | 0 | ≥1 |
| Backups documentados | Sí | No |
| Restore test (último) | <90d | >180d |

---

## Anti-patrones

- Migración `Down()` lanza `NotImplementedException`
- `Up()` borra columna sin backup en `Down()`
- Migración re-fechada manualmente para resolver conflicto merge (timestamps no monotónicos = riesgo de orden incorrecto en RDS)
- FK declarada en convención `WellId` sin `HasForeignKey` explícito
- Sin índice en FK = full table scan en join
- Soft delete inconsistente: unas tablas `IsActive`, otras `DeletedAt`, otras `IsDeleted`
- Audit columns nullable que deberían ser NOT NULL
- DbContext único compartido por todos los módulos (acoplamiento DB)
- Sin global query filter = riesgo de leak cross-tenant
- `Backup` automatizado sin test de `Restore` regular
- BLOB grandes en mismas tablas que datos transaccionales

---

## Multitenancy patterns

### Discriminator column (Single DB)
```csharp
modelBuilder.Entity<Well>()
    .HasQueryFilter(w => w.OperatorId == _currentUser.OperatorId);
```

### Schema per tenant
```csharp
modelBuilder.HasDefaultSchema($"tenant_{tenantId}");
```

### Database per tenant
- Total isolation, mayor costo operacional

### Hybrid (recomendado para fiscalización)
- Operadora ve solo lo suyo (filter)
- Usuario ANH cross-tenant (`IgnoreQueryFilters` + `[RequireTenantType(TenantType.Anh)]`)
- Auditoría obligatoria de acceso cross-tenant

---

## Audit columns recomendadas

```csharp
public abstract class AuditableEntity
{
    public DateTimeOffset CreatedAt { get; private set; }
    public Guid CreatedByUserId { get; private set; }
    public DateTimeOffset UpdatedAt { get; private set; }
    public Guid? UpdatedByUserId { get; private set; }
    public bool IsActive { get; private set; } = true;  // soft delete
}

// Interceptor automático
public class AuditSaveChangesInterceptor : SaveChangesInterceptor
{
    public override ValueTask<InterceptionResult<int>> SavingChangesAsync(...)
    {
        foreach (var entry in eventData.Context.ChangeTracker.Entries<AuditableEntity>())
        {
            if (entry.State == EntityState.Added)
                entry.Entity.CreatedAt = DateTimeOffset.UtcNow;
            entry.Entity.UpdatedAt = DateTimeOffset.UtcNow;
        }
        return base.SavingChangesAsync(...);
    }
}
```

---

## Migration safety checklist

- [ ] `Down()` ejecuta sin error (test en CI)
- [ ] Adición de columna NOT NULL → DEFAULT inline para data existente
- [ ] Eliminación de columna → migración previa la dejó nullable + dejar de leer en código
- [ ] Rename columna → 2 migraciones (add new, copy, drop old)
- [ ] Cambio tipo → en 2 migraciones con shadow column
- [ ] Migración aplicada en staging antes de prod
- [ ] Backup pre-migración automático
- [ ] Tiempo migración estimado (locks tabla > 5 min = riesgo)
- [ ] Compatibilidad backward — código viejo funciona post-migración
- [ ] Test de migración con dataset realista de tamaño

---

## Backup strategy ejemplo

| Tipo | Frecuencia | Retención | Test restore |
|---|---|---|---|
| Full | Diario | 30 días | Mensual |
| Differential | Cada 6h | 7 días | Trimestral |
| Transaction log | Cada 15min | 24h | Quincenal |
| Snapshot pre-deploy | On-demand | 7 días | — |
| Archive (compliance) | Mensual | 7 años | Anual |

---

## Referencias cruzadas

- Multitenancy enforcement → [§02 Arquitectura](02-arquitectura.md)
- Audit log de cambios → [§06 Seguridad](06-seguridad.md)
- Performance índices → [§12 Performance](12-performance.md)
