---
name: animation-effects
description: Create CSS and JavaScript animations, scroll-triggered effects, micro-interactions, loading animations, page transitions, and visual motion design. Use this skill whenever the user asks for animations, transitions, motion effects, or anything that moves on screen. Triggers on: "animate", "animación", "scroll effect", "hover effect", "micro-interaction", "loading animation", "fade in", "parallax", "efecto de scroll", "transición", "partículas", "motion", or any request to add life/movement to an interface. Also use this skill when a landing page or component request specifically emphasizes visual richness or "wow" factor.
---

# Animation & Effects Skill

Builds CSS and JS animations — from subtle micro-interactions to full motion design sequences.

## Animation Categories

### 1. Entrance Animations (load & scroll reveal)
Elements that animate in when the page loads or when they enter the viewport.

### 2. Micro-interactions
Hover states, button feedback, focus rings, toggle transitions — small moments that feel polished.

### 3. Scroll Effects
Parallax, scroll-triggered reveals, sticky elements with transitions, progress indicators.

### 4. Loading States
Spinners, skeletons, progress bars, pulse effects.

### 5. Background Effects
Particle systems, gradient shifts, noise textures, animated SVG patterns.

### 6. Page Transitions
Fade, slide, or morph between views/routes.

---

## Implementation Patterns

### Scroll Reveal (IntersectionObserver — preferred)
```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('is-visible');
      observer.unobserve(entry.target); // animate once
    }
  });
}, { threshold: 0.15 });

document.querySelectorAll('[data-reveal]').forEach(el => observer.observe(el));
```

```css
[data-reveal] {
  opacity: 0;
  transform: translateY(24px);
  transition: opacity 0.6s ease, transform 0.6s ease;
}
[data-reveal].is-visible {
  opacity: 1;
  transform: translateY(0);
}

/* Staggered children */
[data-reveal-group] > * {
  opacity: 0;
  transform: translateY(16px);
  transition: opacity 0.5s ease, transform 0.5s ease;
}
[data-reveal-group].is-visible > *:nth-child(1) { transition-delay: 0ms; }
[data-reveal-group].is-visible > *:nth-child(2) { transition-delay: 80ms; }
[data-reveal-group].is-visible > *:nth-child(3) { transition-delay: 160ms; }
[data-reveal-group].is-visible > *:nth-child(4) { transition-delay: 240ms; }
```

### Staggered Hero Animation (CSS only)
```css
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}

.hero-tag     { animation: fadeUp 0.5s ease both; animation-delay: 0ms; }
.hero-title   { animation: fadeUp 0.6s ease both; animation-delay: 100ms; }
.hero-subtitle{ animation: fadeUp 0.6s ease both; animation-delay: 200ms; }
.hero-cta     { animation: fadeUp 0.5s ease both; animation-delay: 320ms; }
```

### Hover Lift (card/button)
```css
.card {
  transform: translateY(0);
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  transition: transform 0.25s ease, box-shadow 0.25s ease;
}
.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 32px rgba(0,0,0,0.14);
}
```

### Button Press Feedback
```css
.btn {
  transition: transform 0.1s ease, background 0.2s ease;
}
.btn:active {
  transform: scale(0.97);
}
```

### Parallax (lightweight, no library)
```js
window.addEventListener('scroll', () => {
  const scrollY = window.scrollY;
  document.querySelector('.parallax-bg').style.transform = 
    `translateY(${scrollY * 0.4}px)`;
});
```

### Typewriter Effect (CSS)
```css
@keyframes typing {
  from { width: 0 }
  to   { width: 100% }
}
@keyframes blink {
  50% { border-color: transparent }
}

.typewriter {
  overflow: hidden;
  white-space: nowrap;
  border-right: 2px solid currentColor;
  width: 0;
  animation: 
    typing 2.5s steps(30) 0.5s forwards,
    blink 0.75s step-end infinite;
}
```

### CSS Gradient Animation (background)
```css
@keyframes gradientShift {
  0%   { background-position: 0% 50%; }
  50%  { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.animated-bg {
  background: linear-gradient(135deg, #0a0a0a, #1a1a2e, #0d0d25, #050510);
  background-size: 300% 300%;
  animation: gradientShift 8s ease infinite;
}
```

### Skeleton Loader (pulse)
```css
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50%       { opacity: 0.4; }
}

.skeleton {
  background: #e0e0e0;
  border-radius: 4px;
  animation: pulse 1.5s ease-in-out infinite;
}
```

### Counter Animation (JS)
```js
function animateCounter(el, target, duration = 1500) {
  const start = performance.now();
  const update = (now) => {
    const progress = Math.min((now - start) / duration, 1);
    const eased = 1 - Math.pow(1 - progress, 3); // ease-out cubic
    el.textContent = Math.floor(eased * target).toLocaleString();
    if (progress < 1) requestAnimationFrame(update);
  };
  requestAnimationFrame(update);
}
```

---

## Performance Rules

1. **GPU-only properties**: Animate only `transform` and `opacity` — never `width`, `height`, `top`, `left`, `margin`
2. **will-change**: Add `will-change: transform` to elements you KNOW will animate — don't overuse
3. **Respect motion preferences**:
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```
4. **RAF for JS animations**: Always use `requestAnimationFrame`, never `setInterval`
5. **Cleanup**: Disconnect IntersectionObservers when no longer needed

---

## Timing Reference

| Use case | Duration | Easing |
|---|---|---|
| Micro-interaction (button, hover) | 100–200ms | ease |
| UI element open/close | 200–300ms | ease-out |
| Entrance animation | 400–600ms | ease |
| Scroll reveal | 500–700ms | ease |
| Background/ambient | 4000ms+ | ease-in-out |
| Stagger offset between siblings | 60–100ms | — |

---

## Delivery Format

1. Drop-in CSS first (if CSS-only solution exists)
2. Add minimal JS only when CSS alone is insufficient
3. Label sections: `/* === ANIMATION: scroll-reveal === */`
4. Include the `prefers-reduced-motion` block always
