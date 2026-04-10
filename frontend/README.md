# Frontend Skills

Skills para construir interfaces web — desde componentes individuales hasta páginas completas con animaciones.

## Skills disponibles

### `landing-page-builder`
Páginas completas de marketing, portfolios y sitios de producto. Incluye guías de estética, estructura de secciones, paletas de color por dirección visual, y estándares de código.

**Triggerear cuando:** el usuario pide una landing page, home page, sitio web, o cualquier página completa.

**Archivos de referencia:**
- `references/color-palettes.md` — Paletas CSS listas para usar por estética

---

### `ui-components`
Componentes UI aislados y reutilizables: cards, modals, navbars, toasts, tabs, etc. Vanilla o React.

**Triggerear cuando:** el usuario pide un componente específico, no una página completa.

**Catálogo incluido:** Cards · Navbars · Modals/Drawers · Toasts · Forms · Accordions · Badges · Tabs · Skeletons

---

### `animation-effects`
CSS y JS para animaciones: scroll reveal, micro-interactions, parallax, efectos de fondo, contadores animados, typewriter.

**Triggerear cuando:** el usuario pide animaciones, transiciones, efectos de scroll, o quiere que algo "se vea vivo".

**Patrones incluidos:** IntersectionObserver reveal · Staggered entrance · Hover lift · Gradient animation · Skeleton pulse · Counter animation · Typewriter CSS

---

## Principios Generales

- **Sin "AI slop"**: prohibido Inter + gradiente morado + layout genérico
- **Estética con intención**: cada proyecto debe tener una dirección visual clara
- **Performance first**: animar solo `transform` y `opacity`
- **Accesibilidad**: siempre `prefers-reduced-motion`, roles ARIA, contraste suficiente
- **Código completo**: nunca truncar la entrega
