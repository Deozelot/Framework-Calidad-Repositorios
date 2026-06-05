---
description: Análisis de seguridad enfocado (§06 seguridad + §13 dependencias). Arg opcional: [path]
---

Ejecuta un análisis de SEGURIDAD enfocado.

Argumentos recibidos: `$ARGUMENTS`
- Primer token = `path` del repo a analizar (opcional, default = directorio actual).

Usa la skill `quality-framework` en modo `security` con ese argumento. Evalúa las dimensiones
06 (seguridad: secrets, OWASP, auth, cifrado, supply chain) y 13 (dependencias y riesgo de
cadena de suministro). Corre las herramientas de seguridad disponibles (p.ej. gitleaks, audit
de dependencias) y marca cada hallazgo con su fuente. Escribe `security-report.md` en
`./quality-report/YYYY-MM-DD/` del repo analizado y resume en chat los hallazgos y su ruta.
