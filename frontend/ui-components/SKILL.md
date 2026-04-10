---
name: ui-components
description: Build individual, reusable UI components with multiple style variants and production-quality code. Use this skill whenever the user asks for a specific UI element: card, modal, dialog, navbar, sidebar, button group, badge, tooltip, dropdown, accordion, tabs, form fields, avatar, notification toast, data table, or any other isolated interface component. Triggers on phrases like "create a card component", "hazme un modal", "build a navbar", "need a reusable X", or when the user describes a single interface element rather than a full page. Always use this skill for component-level requests.
---

# UI Components Skill

Builds isolated, reusable UI components — delivered as vanilla HTML/CSS/JS snippets or React components depending on context.

## Decision: Vanilla vs React

| Use **Vanilla** when... | Use **React** when... |
|---|---|
| Copy-paste snippet is enough | User specifies React |
| No build environment | Component needs internal state |
| Static display only | Part of a larger React app |

---

## Core Principles

1. **Self-contained**: Each component should work standalone with zero dependencies (vanilla) or minimal imports (React)
2. **Variant-aware**: Always offer at least 2 visual variants (default + 1 alternate style)
3. **Responsive by default**: Works on mobile without extra work
4. **Accessible**: ARIA roles, keyboard navigation, focus states
5. **Documented**: Include usage comments in the code

---

## Component Catalog

### Cards
- **Content card**: image + title + body + CTA
- **Stat card**: metric + label + trend indicator  
- **Profile card**: avatar + name + bio + social links
- **Pricing card**: tier + price + features list + CTA

### Navigation
- **Navbar**: logo + links + CTA; sticky variant; mobile hamburger
- **Sidebar**: icon + label list; collapsible variant
- **Breadcrumb**: chevron-separated path
- **Tabs**: horizontal or vertical; underline or pill variant

### Overlays
- **Modal/Dialog**: backdrop + container + close; trap focus on open
- **Drawer**: slides from left/right; overlay backdrop
- **Tooltip**: hover-triggered; positioning aware
- **Dropdown menu**: trigger button + floating list

### Feedback
- **Toast notification**: top-right, auto-dismiss, variants: success/error/info/warning
- **Alert banner**: inline dismissable message
- **Loading skeleton**: animated placeholder for content
- **Progress bar**: determinate and indeterminate variants

### Forms
- **Input field**: label + input + helper text + error state
- **Select/Dropdown**: custom styled, accessible
- **Toggle/Switch**: animated, with label
- **Checkbox & Radio**: custom styled with group support

### Display
- **Badge/Tag**: status indicators, removable variant
- **Avatar**: image + fallback initials + size variants
- **Accordion**: expand/collapse with animation
- **Data table**: sortable headers, striped rows, pagination

---

## Code Standards

### CSS Architecture (Vanilla)
```css
/* Component namespace to avoid conflicts */
.c-card { ... }
.c-card__image { ... }
.c-card__body { ... }
.c-card--featured { ... }  /* modifier */
```

Use BEM naming: `.c-[component]__[element]--[modifier]`

### State Management (Vanilla JS)
```js
// Data attributes for state
element.dataset.state = 'open' | 'closed'
// CSS responds via [data-state="open"] selector
```

### React Component Pattern
```jsx
// Always: typed props, default values, forwardRef when needed
const Card = ({ title, body, variant = 'default', className = '', ...props }) => {
  return (
    <div className={`card card--${variant} ${className}`} {...props}>
      <h3 className="card__title">{title}</h3>
      <p className="card__body">{body}</p>
    </div>
  );
};

export default Card;
```

### Animation
- Open/close: CSS `transition` on `opacity` + `transform`
- Always set `transition` on both states (open AND closed)
- Avoid `display: none` mid-transition — use `visibility` + `opacity`

```css
.c-modal {
  opacity: 0;
  visibility: hidden;
  transform: translateY(8px);
  transition: opacity 200ms ease, transform 200ms ease, visibility 200ms;
}
.c-modal[data-state="open"] {
  opacity: 1;
  visibility: visible;
  transform: translateY(0);
}
```

### Accessibility Checklist
- [ ] Semantic HTML element (button, nav, dialog, etc.)
- [ ] ARIA roles where needed (`role="dialog"`, `aria-expanded`, `aria-label`)
- [ ] Keyboard navigation (Tab, Enter, Escape)
- [ ] Focus trap in modals/drawers
- [ ] Color contrast ≥ 4.5:1

---

## Delivery Format

For each component, deliver:
1. **The component code** — complete, no truncation
2. **Usage example** — show it rendered with sample data
3. **Variant note** — briefly describe available variants and how to switch

If delivering multiple components, separate with clear `---` dividers and component name headers.
