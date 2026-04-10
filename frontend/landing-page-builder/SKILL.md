---
name: landing-page-builder
description: Build complete, production-quality landing pages and multi-section websites. Use this skill whenever the user asks to create a landing page, home page, hero section, marketing site, portfolio site, product page, or any full-page layout. Triggers on keywords like "landing page", "página de inicio", "home page", "sitio web", "hero section", "make me a site for X", or any request for a complete web page — even if phrased casually. Always use this skill for full-page requests, not just for individual components.
---

# Landing Page Builder

Builds complete, visually distinctive landing pages — HTML/CSS/JS vanilla or React depending on context.

## Decision: Vanilla vs React

| Use **Vanilla HTML/CSS/JS** when... | Use **React** when... |
|---|---|
| Static marketing site | Needs state (toggles, tabs, forms) |
| Pure presentation, no interactivity | User specifies React |
| Fast delivery, no build step | Dashboard-like sections |
| Simple portfolio or promo page | Reusable component system needed |

Default to **vanilla** unless the user specifies React or the design requires dynamic state.

---

## Phase 1: Understand Context Before Writing a Single Line

Ask yourself (or the user if unclear):
- **Product/Service**: What is being sold or presented?
- **Audience**: Who is the visitor? (devs, consumers, businesses, etc.)
- **Goal**: What should the visitor DO? (sign up, contact, buy, learn)
- **Tone**: Professional, playful, luxurious, bold, minimal?
- **Language**: Spanish, English, bilingual?

If the user provides enough context, proceed directly. Otherwise ask ONE focused question.

---

## Phase 2: Choose an Aesthetic Direction

Commit to a bold, specific aesthetic before writing code. Pick one and execute it fully:

- **Neo-brutalist**: Raw borders, stark contrast, exposed grid, bold type
- **Editorial/Magazine**: Asymmetric layout, large type, white space, art-direction
- **Dark luxury**: Deep backgrounds, gold/cream accents, refined spacing
- **Organic/Natural**: Earthy tones, flowing shapes, serif headlines
- **Minimal Swiss**: Grid-strict, functional, limited palette, Helvetica energy
- **Retro-futuristic**: Neon, scanlines, terminal vibes, Y2K or 80s inspired
- **Soft pastel**: Rounded, friendly, airy — for consumer/lifestyle products

**NEVER** default to: purple gradient on white, Inter font everywhere, generic SaaS aesthetic, or cookie-cutter layouts.

---

## Phase 3: Page Structure

Build these sections in order, adapting to the project:

### Required Sections
1. **Navbar** — Logo + nav links + CTA button. Sticky on scroll.
2. **Hero** — Headline, subheadline, primary CTA, visual element (illustration, mockup, or background effect)
3. **Footer** — Links, social, copyright

### Optional Sections (include what fits)
4. **Features/Benefits** — 3–6 cards or icon-list rows
5. **Social Proof** — Testimonials, logos, stats counter
6. **How It Works** — Step-by-step (numbered or timeline)
7. **Pricing** — 2–3 tier cards with highlight on recommended
8. **FAQ** — Accordion or simple list
9. **Final CTA** — Repeat the main conversion action

---

## Phase 4: Code Quality Standards

### Typography
- Use Google Fonts — pick fonts that match the aesthetic
- Heading: 1 display/serif/slab font
- Body: 1 clean readable font
- **Forbidden**: Arial, Roboto, Inter (unless intentional)

### Color
- Define all colors as CSS custom properties at `:root`
- Max 4 colors: background, foreground, accent, muted
- Accent should be bold and intentional

### CSS
- Use CSS custom properties (`--color-accent`, `--font-heading`, etc.)
- Mobile-first with `@media (min-width: ...)` breakpoints
- Flexbox and CSS Grid — no floats
- Smooth scroll: `scroll-behavior: smooth` on `html`

### Navbar behavior (vanilla JS)
```js
// Sticky navbar with shadow on scroll
window.addEventListener('scroll', () => {
  navbar.classList.toggle('scrolled', window.scrollY > 50);
});
```

### Animations
- Use CSS `@keyframes` + `animation` for load effects
- Use `IntersectionObserver` for scroll-reveal (see animation-effects skill)
- Keep transitions under 400ms for UI feedback

### Accessibility
- Semantic HTML: `<header>`, `<main>`, `<section>`, `<footer>`, `<nav>`
- `alt` text on all images
- Sufficient color contrast (WCAG AA minimum)
- Focus styles visible on interactive elements

---

## Phase 5: Delivery

1. Output the complete file(s) — never truncate
2. For vanilla: single `index.html` with inline `<style>` and `<script>`, OR separate files if user prefers
3. For React: JSX component(s) with Tailwind or CSS modules
4. Add a short comment block at the top: `<!-- Landing Page: [Project Name] | Stack: [HTML/React] -->`

---

## Reference Files

- `references/sections.md` — Copy-ready HTML snippets for each section type
- `references/color-palettes.md` — Curated palettes per aesthetic direction

---

## Examples of Good Trigger Prompts

- "Hazme una landing page para mi app de tours en Jardín Antioquia"
- "Create a portfolio site for a backend developer"
- "Build a marketing page for a SaaS product called Squads"
- "Dame un hero section moderno con animación"
