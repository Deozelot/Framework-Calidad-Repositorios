# framework-calidad — Plugin de Claude Code

Ejecuta el *Framework de Criterios — Análisis de Calidad de Repositorio v1.0*
(216 criterios, 23 dimensiones) sobre cualquier repositorio. Medición híbrida:
corre herramientas reales si están disponibles, usa el juicio de Claude leyendo
el código cuando no. Genera reportes en `./quality-report/YYYY-MM-DD/` del repo analizado.

## Instalación (marketplace local)

En Claude Code, desde la raíz de este repositorio:

```
/plugin marketplace add .
/plugin install framework-calidad@framework-calidad-marketplace
```

## Comandos

| Comando | Qué hace | Salida |
|---------|----------|--------|
| `/quality-quick [path]` | Subset por-PR (§05, §06, §07, §10) | `quick-report.md` |
| `/quality-full [path] [tipo]` | 216 criterios + score global. `tipo` = critico\|mvp\|legacy (default mvp) | `informe-completo.md` + `score.md` |
| `/quality-security [path]` | Seguridad (§06 + §13) | `security-report.md` |
| `/quality-deep-dive [path] "hallazgo"` | Causa raíz + arreglos P1/P2/P3 (§25 + §26) | `deep-dive-<slug>.md` + `arreglos.md` |

`path` es opcional (default = directorio actual). Cada hallazgo marca su fuente:
🔧 medido (herramienta) vs 🧠 estimado (juicio de Claude).
