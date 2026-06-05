# §15 — Frontend (Angular/React/Vue)

[← Volver al índice](INDEX.md) · Anterior: [§14](14-datos-persistencia.md) · Siguiente: [§16](16-backend.md)

---

## Objetivo

Criterios específicos para apps frontend SPA: design system, state management, accesibilidad, performance.

---

## Criterios

| # | Criterio | Cómo medir |
|---|---|---|
| 15.1 | Component design system (UIKit) | Estructura atoms/molecules/organisms |
| 15.2 | OnPush change detection (Angular) | Audit `ChangeDetectionStrategy` |
| 15.3 | Standalone components (Angular 14+) | Audit `standalone: true` |
| 15.4 | State management coherente (NgRx/Signals/Redux) | Política documentada + audit |
| 15.5 | Accessibility WCAG 2.1 AA | Audit con `axe-core`, lighthouse |
| 15.6 | i18n configurado | ngx-translate, i18next, locale files |
| 15.7 | Theming/dark mode | Tokens design system |
| 15.8 | Responsive breakpoints documentados | Tailwind/CSS vars |
| 15.9 | Error boundaries / global error handler | Audit |
| 15.10 | Lazy loading + code splitting | Bundle analysis |
| 15.11 | Service workers / PWA | Si aplica |
| 15.12 | Performance budgets | `angular.json` budgets / Lighthouse CI |

---

## Comandos de referencia

```bash
# Standalone components Angular
grep -rn "standalone: true" src --include="*.ts" | wc -l
grep -rn "@Component\|@NgModule" src --include="*.ts"

# OnPush usage
grep -rn "ChangeDetectionStrategy.OnPush" src --include="*.ts" | wc -l
grep -rn "@Component" src --include="*.ts" | wc -l

# State management mix
find src -name "*.actions.ts" | wc -l       # NgRx actions
find src -name "*.effects.ts" | wc -l       # NgRx effects
grep -rn "signal<\|computed(\|effect(" src --include="*.ts" | wc -l  # Signals

# A11y audit
npx axe https://localhost:4200 --tags wcag2a,wcag2aa
npx lighthouse https://localhost:4200 --only-categories=accessibility

# i18n keys
find src -name "*.json" -path "*i18n*" | head
find src -name "locale.ts" | wc -l

# Lazy loading routes
grep -rn "loadChildren\|loadComponent" src --include="*.ts" | wc -l

# Performance budgets
grep -A10 '"budgets"' angular.json

# Bundle analysis
ng build --stats-json
npx webpack-bundle-analyzer dist/<app>/stats.json
```

---

## Evidencia esperada

- Conteo standalone vs module-based components
- Conteo OnPush vs Default change detection
- Política NgRx/Signals documentada
- Axe-core report con violaciones WCAG por severidad
- Lista locales i18n soportados
- Bundle size por chunk principal + lazy chunks
- Lighthouse score (performance/accessibility/best practices/SEO)

---

## Umbrales

| Métrica | Verde | Rojo |
|---|---|---|
| Standalone components | 100% nuevos | <50% |
| OnPush ChangeDetection | >80% | <50% |
| WCAG 2.1 AA violations | 0 críticas | ≥5 críticas |
| Lighthouse accessibility | ≥95 | <80 |
| Lighthouse performance | ≥85 | <60 |
| Bundle main inicial gzip | <500KB | >2MB |
| Lazy chunks por route | <300KB | >1MB |
| FCP (First Contentful Paint) | <1.5s | >3s |
| LCP (Largest Contentful Paint) | <2.5s | >4s |
| CLS (Cumulative Layout Shift) | <0.1 | >0.25 |

---

## Atomic Design (recomendado UIKit)

```
shared/ui/components/
├── atoms/          (button, input, badge, icon, ...)
├── molecules/      (form-field, search-bar, ...)
├── organisms/      (header, sidebar, table, ...)
├── templates/      (layouts genéricos)
└── pages/          (páginas completas reutilizables)
```

Reglas:
- Atoms NO importan molecules/organisms
- Molecules pueden importar atoms
- Organisms pueden importar molecules + atoms
- Pages componen organisms

---

## State management decision tree

```
¿Estado solo dentro del componente?
  → Signal local

¿Estado compartido entre componentes hermanos?
  → Signal en parent + input/output, o Service con signal

¿Estado compartido entre features no hermanas?
  → NgRx (action + reducer + selector + effect)

¿Estado paginado con filtros server-side?
  → NgRx (managed lifecycle complejo)

¿Estado puramente cliente (UI: open/closed, hover)?
  → Signal local
```

---

## Anti-patrones

- ChangeDetection default + Observable pipe (re-renders innecesarios)
- `console.log` en producción
- `subscribe()` sin `unsubscribe()` o `takeUntilDestroyed()`
- Lógica de negocio en componente vs en service/store
- Routing sin `loadChildren` (todo en bundle main)
- Sin `OnPush` en componentes que reciben muchos props
- Bindings inseguros `[innerHTML]` sin sanitizar
- `any` types proliferando
- CSS inline en lugar de tokens design system
- A11y ignorado: sin labels, sin `aria-*`, contraste insuficiente
- i18n hardcodeado: `<button>Guardar</button>` en lugar de `{{ 'common.save' | translate }}`
- Imports `* as X` (rompen tree shaking)
- Forms reactivos sin tipado (`new FormGroup({})` sin type param)
- Signals + RxJS mezclados sin razón clara

---

## WCAG 2.1 AA — checklist mínima

- Contraste texto/fondo ≥4.5:1 (≥3:1 texto grande)
- Todos inputs con `<label for="...">`
- Botones con texto o `aria-label`
- Imágenes con `alt` (descriptivo o `alt=""` si decorativa)
- Form errors anunciados con `aria-live="polite"`
- Navegación por teclado completa (tab, enter, escape)
- Focus visible en todos elementos interactivos
- Skip links al main content
- Heading hierarchy correcta (h1→h2→h3)
- `lang` atribute en `<html>`
- Modal con focus trap
- Loading states anunciados a screen readers

---

## Angular 21 specific

- `inject()` sobre constructor injection (composability)
- Signals para state local
- `computed()` para derived state
- `effect()` para side effects (cuidado con loops)
- Control flow nuevo: `@if`, `@for`, `@switch` (mejor que `*ngIf`)
- `provideHttpClient(withInterceptors([...]))` (functional)
- Defer view templates: `@defer (on viewport) { ... }`

---

## Referencias cruzadas

- Performance Lighthouse → [§12 Performance](12-performance.md)
- TypeScript strict + any → [§05 Buenas prácticas](05-buenas-practicas-codigo.md)
- Tests unitarios componentes → [§07 Testing](07-testing-cobertura.md)
- Storybook docs → [§09 Documentación](09-documentacion.md)
