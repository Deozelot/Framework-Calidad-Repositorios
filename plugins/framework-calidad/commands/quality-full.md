---
description: Análisis de calidad completo (216 criterios, 23 dimensiones) + score global. Args opcionales: [path] [tipo: critico|mvp|legacy]
---

Ejecuta un análisis de calidad COMPLETO del repositorio.

Argumentos recibidos: `$ARGUMENTS`
- Primer token = `path` del repo a analizar (opcional, default = directorio actual).
- Segundo token = `tipo` de proyecto para la matriz de pesos: `critico`, `mvp` o `legacy` (default `mvp`).

Usa la skill `quality-framework` en modo `full` con esos argumentos. Sigue su flujo de
7 pasos: detectar repo y stack, inventariar herramientas, medir, evaluar los criterios
de las dimensiones 01–18/23/24, scorear con la matriz de pesos del `tipo`, y escribir
`informe-completo.md` + `score.md` en `./quality-report/YYYY-MM-DD/` del repo analizado.
Termina con un resumen en chat: score global, top hallazgos y la ruta de los archivos.
