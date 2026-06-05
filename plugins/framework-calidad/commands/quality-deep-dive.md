---
description: Investigación a fondo de un hallazgo (§25 análisis + §26 arreglos). Args: [path] "descripción del hallazgo"
---

Ejecuta una investigación EN PROFUNDIDAD sobre un hallazgo específico.

Argumentos recibidos: `$ARGUMENTS`
- Si el primer token es una ruta existente, trátalo como `path` del repo (default = directorio actual).
- El texto entre comillas (o el resto del argumento) = `hallazgo` a investigar. Es obligatorio;
  si falta, pídelo antes de continuar.

Usa la skill `quality-framework` en modo `deep-dive` con esos argumentos. Sigue las plantillas
B (análisis en profundidad, §25) y C (arreglos priorizados, §26) de `references/plantillas.md`:
analiza causa raíz del hallazgo leyendo el código y datos de herramientas, luego propón arreglos
priorizados P1/P2/P3. Escribe `deep-dive-<slug>.md` y `arreglos.md` en `./quality-report/YYYY-MM-DD/`
del repo analizado (`<slug>` derivado del hallazgo). Resume en chat la causa raíz, los arreglos
y la ruta de los archivos.
