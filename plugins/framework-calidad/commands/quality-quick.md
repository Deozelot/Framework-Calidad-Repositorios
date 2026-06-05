---
description: Análisis rápido por-PR (subset de alto impacto). Arg opcional: [path]
---

Ejecuta un análisis de calidad RÁPIDO, pensado para revisión por PR.

Argumentos recibidos: `$ARGUMENTS`
- Primer token = `path` del repo a analizar (opcional, default = directorio actual).

Usa la skill `quality-framework` en modo `quick` con ese argumento. Evalúa solo el subset
de alto impacto (dimensiones 05 buenas prácticas, 06 seguridad/secrets, 07 testing/cobertura,
10 procesos/trazabilidad). Escribe `quick-report.md` en `./quality-report/YYYY-MM-DD/` del
repo analizado y termina con un resumen en chat: flags encontrados y ruta del archivo.
