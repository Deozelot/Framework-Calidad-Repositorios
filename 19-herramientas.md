# §19 — Herramientas Recomendadas por Categoría

[← Volver al índice](INDEX.md) · Anterior: [§18](18-metricas-cuantitativas.md) · Siguiente: [§20](20-plantilla-informe.md)

---

## Objetivo

Mapeo de herramientas por categoría/stack para automatizar la captura de métricas y aplicación de criterios.

---

## Matriz de herramientas

| Categoría | .NET | TypeScript/Node | Universal |
|---|---|---|---|
| **Static analysis** | Roslyn, SonarQube, NDepend | SonarQube, ESLint | CodeClimate |
| **Architecture rules** | NetArchTest | dependency-cruiser, madge | ArchUnit |
| **Coverage** | coverlet + ReportGenerator | vitest --coverage, nyc, c8 | Codecov, SonarCloud |
| **Security scan** | dotnet list --vulnerable, Snyk | npm audit, Snyk | Gitleaks, TruffleHog, Trivy |
| **Mutation testing** | Stryker.NET | StrykerJS | — |
| **Linting** | Roslyn analyzers, StyleCop | ESLint, Prettier | EditorConfig |
| **Dependency mgmt** | Dependabot | Dependabot, Renovate | Renovate |
| **Bundle analysis** | — | webpack-bundle-analyzer, Source Map Explorer | — |
| **Performance** | BenchmarkDotNet | clinic.js, autocannon | k6, JMeter |
| **Documentation** | DocFX, Swashbuckle | Compodoc, TypeDoc | MkDocs, Docusaurus |
| **Visual regression** | — | Chromatic, Percy | BackstopJS |
| **E2E** | — | Playwright, Cypress | Selenium |
| **Log aggregation** | Serilog + Seq | Pino + Loki | Datadog, Elastic |
| **Observability** | OpenTelemetry .NET | OpenTelemetry JS | Grafana, Honeycomb |

---

## Detalle por categoría

### Static analysis

| Tool | Fortalezas | Pricing |
|---|---|---|
| **SonarQube/SonarCloud** | Multi-lenguaje, code smells, security hotspots | Free OSS / paid org |
| **NDepend** | Métricas .NET avanzadas (LCOM, depth, fan-in/out) | Comercial caro |
| **Roslyn Analyzers** | Built-in .NET, custom rules | Gratis |
| **CodeClimate** | Trend tracking, gamification | Free OSS / paid |
| **DeepSource** | Auto-fix sugerencias | Free OSS / paid |

### Architecture testing

| Tool | Stack |
|---|---|
| **NetArchTest** | .NET — fluent API para reglas |
| **ArchUnit** | Java + .NET port |
| **dependency-cruiser** | Node/TS — reglas dependencias |
| **madge** | TS — circular dependencies |
| **structurizr** | C4 model as code |

### Coverage

| Tool | Stack | Output |
|---|---|---|
| **coverlet** | .NET | Cobertura/OpenCover/JSON |
| **ReportGenerator** | .NET | HTML reports |
| **vitest** | Vite/TS | V8 native coverage |
| **nyc/c8** | Node | V8 coverage |
| **Codecov** | Universal | Cloud reports, PR comments |
| **Coveralls** | Universal | Similar |
| **SonarCloud** | Universal | Incluye coverage en otras métricas |

### Security

| Tool | Qué hace |
|---|---|
| **Gitleaks** | Detecta secrets en repo + historia |
| **TruffleHog** | Similar, más patterns |
| **Snyk** | Vulnerabilidades deps + CVE + container |
| **OWASP Dependency-Check** | CVE scan multi-language |
| **Trivy** | Container + IaC + deps |
| **OWASP ZAP** | DAST scan apps web |
| **Burp Suite** | Pentesting manual |
| **Semgrep** | SAST con custom rules |

### Mutation testing

| Tool | Stack |
|---|---|
| **Stryker.NET** | .NET — generate mutants |
| **StrykerJS** | TS/Node |
| **PIT** | Java |

### Linting / Formatting

| Tool | Stack | Notas |
|---|---|---|
| **ESLint** | TS/JS | Reglas extensas + plugins Angular/React |
| **Prettier** | TS/JS/CSS/MD | Solo formato, no semántica |
| **Roslyn Analyzers** | .NET | Built-in + Microsoft.CodeAnalysis |
| **StyleCop** | .NET | Convenciones C# clásicas |
| **dotnet-format** | .NET | Aplicación formato |
| **EditorConfig** | Universal | Reglas comunes IDE |
| **commitlint** | Git | Validar Conventional Commits |
| **markdownlint** | MD | Linting docs |

### Performance

| Tool | Stack | Tipo |
|---|---|---|
| **BenchmarkDotNet** | .NET | Micro-benchmarks |
| **k6** | Universal | Load testing scripting JS |
| **JMeter** | Universal | Load testing GUI |
| **Apache Bench (ab)** | Universal | Quick HTTP benchmark |
| **clinic.js** | Node | Profiling |
| **autocannon** | Node | HTTP benchmark |
| **Lighthouse CI** | Web | Performance budgets |
| **WebPageTest** | Web | Real device testing |

### Testing E2E

| Tool | Stack | Strengths |
|---|---|---|
| **Playwright** | Universal | Multi-browser, fast, modern |
| **Cypress** | Web | Great DX, time-travel debug |
| **Selenium** | Universal | Maduro, lenguaje agnóstico |
| **Puppeteer** | Chromium | Headless control |
| **TestCafe** | Web | No WebDriver |

### Observability stack

| Layer | OSS | SaaS |
|---|---|---|
| Logs | Loki + Grafana | Datadog, Splunk |
| Metrics | Prometheus + Grafana | Datadog, New Relic |
| Tracing | Jaeger, Tempo | Honeycomb, Lightstep |
| APM | OpenTelemetry collector | Datadog APM, Dynatrace |
| Errors | Sentry self-hosted | Sentry SaaS, Rollbar |
| Alerting | Alertmanager + PagerDuty | Opsgenie, PagerDuty |
| Dashboards | Grafana | Datadog, Tableau |

### Documentation

| Tool | Use case |
|---|---|
| **Swashbuckle** | .NET OpenAPI auto-gen |
| **Scalar** | Modern OpenAPI UI |
| **NSwag** | OpenAPI client gen + docs |
| **DocFX** | .NET docs static site |
| **Compodoc** | Angular components docs |
| **TypeDoc** | TS API docs |
| **MkDocs** | Markdown static site |
| **Docusaurus** | React-based docs |
| **PlantUML** | Diagrams as code |
| **Mermaid** | Diagrams inline GitHub |
| **C4-PlantUML** | C4 model templates |
| **Storybook** | Component catalog |
| **Chromatic** | Visual regression Storybook |

### CI/CD platforms

| Tool | Notes |
|---|---|
| **GitHub Actions** | Tight GitHub integration |
| **GitLab CI** | All-in-one |
| **Azure DevOps Pipelines** | Microsoft ecosystem |
| **CircleCI** | Cloud nativa |
| **Jenkins** | Maduro, self-hosted |
| **TeamCity** | JetBrains |
| **Drone CI** | Container-native |
| **Argo CD** | Kubernetes GitOps |
| **Flux** | Kubernetes GitOps |

### Container security

| Tool | Use |
|---|---|
| **Trivy** | Image scan + IaC |
| **Grype** | Image scan |
| **Docker Scout** | Docker-native |
| **Snyk Container** | Multi-source |
| **Anchore** | Image policies |

### Dependency management

| Tool | Auto-PRs |
|---|---|
| **Dependabot** | Sí (GitHub native) |
| **Renovate** | Sí (más config) |
| **Snyk** | Sí + security focus |

---

## Stack recomendado por escala

### Equipo pequeño (1-5 devs)
- GitHub Actions + Codecov + Dependabot
- ESLint + Prettier + dotnet-format
- SonarCloud (free for OSS)
- Sentry (free tier)
- Storybook + Chromatic (free tier)

### Equipo mediano (5-20 devs)
- + SonarQube self-hosted
- + Datadog o New Relic
- + Snyk for security
- + Renovate (más control que Dependabot)
- + k6 load tests en CI

### Empresa (20+ devs)
- + NDepend para arquitectura
- + Stryker mutation testing
- + Custom Semgrep rules
- + Burp Suite pentesting
- + Splunk/Datadog enterprise

---

## Mínimo viable para empezar (gratis)

1. **GitHub Actions** — CI
2. **ESLint + Prettier + Roslyn** — linting/formato
3. **coverlet + vitest** — coverage
4. **Dependabot** — security updates
5. **gitleaks** — secrets scan
6. **NetArchTest + dependency-cruiser** — arch rules
7. **Storybook** — UI catalog
8. **Mermaid en .md** — diagramas
9. **Conventional Commits + commitlint** — commits
10. **Sentry free** — error tracking

Setup ~1 día. Cubre 80% del framework.

---

## Referencias cruzadas

Todas las herramientas aquí mapeadas son referenciadas en sus secciones respectivas (§01 a §17).
