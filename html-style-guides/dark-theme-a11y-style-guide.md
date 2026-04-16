# Dark Theme Accessibility Style Guide

A standalone, general-purpose reference for building modern dark theme UIs that are accessible to users with colorblindness (all major types) and low vision.

---

## 1. Philosophy & Guiding Principles

**Never use color as the sole means of conveying information.**
Every state, status, or category must have at least one non-color differentiator: shape, icon, text label, or pattern.

**Contrast and luminance over hue.**
Dark backgrounds flatten hue-based distinctions. Rely on luminance contrast ratios (WCAG math) rather than "does it look different enough?"

**Design for the extremes; the middle comes for free.**
If your UI works for a user with achromatopsia (no color perception) and a user with 2× zoom at 20/200 vision, it works for everyone else by default.

**The three colorblindness types to design against:**
| Type | Affects | Population |
|---|---|---|
| Deuteranopia | Green receptors | ~5% of males |
| Protanopia | Red receptors | ~1% of males |
| Tritanopia | Blue receptors | Rare |
| Achromatopsia | All color | Very rare |

Red/green pairings fail deuteranopia and protanopia simultaneously — avoid them as sole differentiators in any context.

---

## 2. Color Palette

### Background Tiers

```
#0f172a  — base page background
#1e293b  — surface (cards, panels, modals)
#334155  — raised (dropdowns, tooltips, hover states on surfaces)
```

Use these consistently so elevation is conveyed through luminance, not hue.

### Colorblind-Safe Accent Palette

| Role | Hex | Notes |
|---|---|---|
| Primary / interactive | `#3b82f6` (blue) | Universally distinguishable across all colorblindness types |
| Warning / highlight | `#f59e0b` (amber) | Safe for deuteranopia and protanopia; avoid pairing with green |
| Secondary / info | `#06b6d4` (cyan) | Distinguishable from blue for most colorblindness types |
| Accent / data series | `#a855f7` (purple) | Works well alongside blue and amber |

**Avoid:** Red (`#ef4444`) and green (`#10b981`) as a paired differentiator. Use them individually with icons and labels, never as a red-vs-green comparison.

### Text Colors

| Role | Hex | Contrast on `#0f172a` |
|---|---|---|
| Primary text | `#f8fafc` | ~15.8:1 — passes AAA |
| Secondary / muted | `#94a3b8` | ~4.6:1 — passes AA |
| Disabled / placeholder | `#475569` | ~2.7:1 — use sparingly; never for meaningful content |

Both primary and secondary pass WCAG AA (4.5:1) for body text on the base background.

### Semantic Colors

Semantic colors **must always** be paired with an icon glyph and a text label. Color is a reinforcement, not the signal.

| State | Color | Icon | Label pattern |
|---|---|---|---|
| Error | `#ef4444` | ⚠ | "Error: [message]" |
| Success | `#10b981` | ✓ | "Success: [message]" |
| Warning | `#f59e0b` | ! | "Warning: [message]" |
| Info | `#06b6d4` | ℹ | "Note: [message]" |

### Contrast Ratio Targets

| Context | Minimum ratio | WCAG level |
|---|---|---|
| Body text (≥ 18px) | 4.5:1 | AA |
| Small text (< 18px) | 4.5:1 | AA |
| Critical UI (error messages, form labels) | 7:1 | AAA |
| UI components / graphical objects | 3:1 | AA |
| Disabled states | No requirement | — |

**Verify with:** [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) or the browser DevTools color picker's contrast badge.

---

## 3. Typography

### Font Stack

```css
font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
```

No external font calls. System fonts render with the OS's sub-pixel rendering, improving legibility at low contrast.

### Size Scale (8pt Grid)

```
12px  — captions, fine print
14px  — secondary labels, metadata
16px  — minimum body copy
18px  — recommended body copy
24px  — subheadings (h3/h4)
32px  — section headings (h2)
48px  — page titles (h1)
```

Never go below 16px for meaningful body content. Prefer 18px.

### Line Height & Spacing

```css
/* Body copy */
line-height: 1.6;
letter-spacing: 0.01em;

/* Headings */
line-height: 1.3;
letter-spacing: -0.01em;

/* ALL CAPS labels (badges, overlines) */
letter-spacing: 0.08em;
font-size: 12px;
font-weight: 600;
```

### Weight

- Body: `400`
- Interactive labels, buttons: `600`
- Headings: `600–700`
- **Never use weights below 400 on dark backgrounds.** Thin strokes disappear at low contrast.

---

## 4. Spacing & Layout (8pt Grid)

### Base Values

```
4px   — micro gap (icon-to-label)
8px   — tight spacing (within components)
12px  — compact padding
16px  — standard padding
24px  — comfortable section gap
32px  — large section gap
48px  — page section separation
64px  — major layout zones
```

All spacing values should be multiples of 4 (preferably 8) for alignment consistency.

### Touch & Click Targets

| Context | Minimum size | WCAG criterion |
|---|---|---|
| Touch (mobile) | 44×44px | WCAG 2.5.5 (AAA) |
| Click (desktop) | 32×32px | Practical minimum |
| Spacing between adjacent targets | 8px | Prevents mis-taps |

Ensure the interactive region (padding included) meets minimums, not just the visible icon or label.

### Border Radius

```
4px   — small elements (badges, tags, inline code)
8px   — default (inputs, buttons, dropdowns)
16px  — cards, panels, modals
9999px — pill / fully rounded (toggles)
```

---

## 5. Component Patterns

### Buttons

```css
/* Focus ring — apply to all interactive elements */
:focus-visible {
  outline: 2px solid #3b82f6;
  outline-offset: 2px;
}

/* Primary button */
.btn-primary {
  background: #3b82f6;
  color: #0f172a;          /* dark text on blue — contrast ~4.7:1 */
  font-weight: 600;
  min-height: 44px;
  padding: 10px 20px;
  border-radius: 8px;
  border: none;
}

/* Secondary button */
.btn-secondary {
  background: transparent;
  color: #f8fafc;
  border: 2px solid #334155;
  font-weight: 600;
  min-height: 44px;
  padding: 10px 20px;
  border-radius: 8px;
}
```

**States:**
| State | Visual treatment |
|---|---|
| Default | Base styles above |
| Hover | `filter: brightness(1.1)` |
| Active | `filter: brightness(0.9)` |
| Disabled | `opacity: 0.4; cursor: not-allowed; pointer-events: none` |
| Focus | 2px blue outline, offset 2px |

**Rules:**
- Always include a visible text label. Icon-only buttons require `aria-label` on the button element.
- Never disable pointer events on elements that need keyboard focus — use `aria-disabled` + custom JS instead if you need clickable-but-inactive behavior.

### Form Inputs

```css
.input {
  background: #1e293b;
  border: 2px solid #334155;     /* 2px, not 1px — low vision */
  border-radius: 8px;
  color: #f8fafc;
  font-size: 16px;                /* prevents iOS auto-zoom */
  min-height: 44px;
  padding: 10px 12px;
}

.input:focus-visible {
  outline: 2px solid #3b82f6;
  outline-offset: 2px;
  border-color: #3b82f6;
}

.input[aria-invalid="true"] {
  border-color: #ef4444;
}
```

**Rules:**
- Label always visually above the input — never use placeholder text as the only label.
- Error messages: `border-color` change + ⚠ icon + text `"Error: ..."` below the field.
- Associate label and input: `<label for="id">` or `aria-labelledby`.
- Associate error message: `aria-describedby="error-id"` on the input.

```html
<!-- Correct pattern -->
<label for="email">Email address</label>
<input
  id="email"
  type="email"
  aria-describedby="email-error"
  aria-invalid="true"
/>
<p id="email-error" role="alert">
  <span aria-hidden="true">⚠</span> Error: Enter a valid email address.
</p>
```

### Cards / Panels

```css
.card {
  background: #1e293b;           /* one tier above page bg */
  border-radius: 16px;
  padding: 24px;
  border: 1px solid #334155;
}

/* Category differentiation via accent border + icon+label, not color alone */
.card--warning {
  border-left: 4px solid #f59e0b;
}
```

Cards conveying a category (status, type) must use an icon + text label alongside any color accent. A colored left border alone is not sufficient.

### Status Indicators / Badges

```html
<!-- Correct: color + icon + text -->
<span class="badge badge--success">
  <svg aria-hidden="true"><!-- checkmark --></svg>
  Active
</span>

<!-- Incorrect: color alone -->
<span class="badge badge--success"></span>
```

**Achromatopsia fallback:** Use pattern fills (CSS or SVG) as a tertiary differentiator in dense UIs (dashboards, tables):
- Success: diagonal lines (`/////`)
- Error: crosshatch (`#####`)
- Warning: dots (`. . .`)

### Data Visualization / Charts

**Palette for chart data series:**

```
Series 1: #3b82f6 (blue)
Series 2: #f59e0b (amber)
Series 3: #06b6d4 (cyan)
Series 4: #a855f7 (purple)
Series 5: #f8fafc (white/light) — use only when needed
```

**Rules:**
- Never differentiate series by red vs. green alone.
- Add SVG pattern fills as a secondary differentiator (diagonal stripes, dots, crosshatch).
- Always include: direct data labels on chart elements OR a legend that pairs color swatch + text + pattern.
- Provide a text/table alternative for screen readers (`<table>` beneath the `<canvas>` or `aria-label` describing the trend).

---

## 6. Focus & Keyboard Navigation

```css
/* Global baseline — use focus-visible, not focus, to avoid rings on mouse clicks */
*:focus-visible {
  outline: 2px solid #3b82f6;
  outline-offset: 2px;
  border-radius: 4px;
}

/* NEVER do this */
* { outline: none; }       /* removes all keyboard focus indicators */
*:focus { outline: none; } /* same problem */
```

**Rules:**
- Tab order must follow visual reading order (left-to-right, top-to-bottom).
- Interactive elements within a component group (toolbar, radio group) should use `roving tabindex` — only one tab stop for the group, arrow keys move within.
- Modal dialogs: trap focus inside while open; return focus to the trigger on close.
- Skip-to-content link must be the first focusable element on every page:

```html
<a href="#main-content" class="skip-link">Skip to main content</a>

<style>
.skip-link {
  position: absolute;
  top: -100%;
  left: 8px;
  background: #3b82f6;
  color: #0f172a;
  padding: 8px 16px;
  border-radius: 0 0 8px 8px;
  font-weight: 600;
  z-index: 9999;
}
.skip-link:focus {
  top: 0;
}
</style>
```

---

## 7. Motion & Animation

```css
/* Default transitions */
.interactive {
  transition: background-color 150ms ease, transform 150ms ease;
}

.panel {
  transition: opacity 250ms ease, transform 250ms ease;
}

/* Respect user preference — disable all motion */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

**Rules:**
- No content may flash more than 3 times per second (WCAG 2.3.1 — seizure safety).
- Parallax and large-scale motion should be disabled under `prefers-reduced-motion`.
- Loading spinners: acceptable under reduced motion if they are not flashing; prefer a simple static indicator.
- Auto-playing video: provide pause control; mute by default.

---

## 8. Icons & Visual Language

**Sizing:**
```
20×20px — minimum icon size
24×24px — preferred for inline icons
32×32px — standalone action icons
```

**Stroke weight:** Minimum 2px stroke for outline-style icons. At 20px with a 1px stroke, icons become illegible at low vision or on non-retina displays.

**Pairing rules:**

| Usage | Requirement |
|---|---|
| Icon beside a text label | `aria-hidden="true"` on the icon; label provides the accessible name |
| Icon-only button | `aria-label="[action]"` on the `<button>` |
| Icon conveying status | Must also have adjacent visible text; `aria-hidden="true"` on icon |
| Decorative icon | `aria-hidden="true"` |

```html
<!-- Icon + label: icon is decorative -->
<button>
  <svg aria-hidden="true" focusable="false"><!-- ... --></svg>
  Save changes
</button>

<!-- Icon-only: button carries the label -->
<button aria-label="Close dialog">
  <svg aria-hidden="true" focusable="false"><!-- × --></svg>
</button>
```

---

## 9. ARIA & Semantic HTML Checklist

### Use Semantic HTML First

| Task | Native element | Avoid |
|---|---|---|
| Button | `<button>` | `<div onclick>` |
| Navigation | `<nav>` | `<div role="navigation">` |
| Main content | `<main>` | `<div id="main">` |
| Page regions | `<header>`, `<footer>`, `<aside>` | Generic `<div>`s |
| Lists | `<ul>/<ol>/<li>` | `<div>`s with bullet CSS |
| Tables | `<table>`, `<th scope>`, `<caption>` | CSS grid masquerading as a table |

`role`, `aria-label`, and `aria-describedby` are supplements for cases where native semantics fall short, not replacements.

### Live Regions

```html
<!-- Toast notifications, status updates -->
<div aria-live="polite" aria-atomic="true" class="toast-region">
  <!-- Injected dynamically -->
</div>

<!-- Critical errors that interrupt -->
<div role="alert">
  <!-- aria-live="assertive" is implicit on role="alert" -->
</div>
```

Use `aria-live="polite"` (waits for user to finish current action) for most notifications. Use `role="alert"` only for genuine errors or urgent interruptions.

### Images

```html
<!-- Meaningful image -->
<img src="chart.png" alt="Bar chart showing revenue growth from $2M to $4.5M between 2023 and 2025." />

<!-- Decorative image -->
<img src="background-texture.png" alt="" role="presentation" />

<!-- SVG used as image -->
<svg role="img" aria-label="Company logo">...</svg>
```

### Key ARIA Patterns Reference

| Pattern | Attributes to use |
|---|---|
| Modal dialog | `role="dialog"`, `aria-modal="true"`, `aria-labelledby` |
| Expandable section | `aria-expanded="true/false"` on trigger |
| Tab panel | `role="tablist"`, `role="tab"`, `role="tabpanel"`, `aria-selected`, `aria-controls` |
| Progress bar | `role="progressbar"`, `aria-valuenow`, `aria-valuemin`, `aria-valuemax` |
| Combobox / autocomplete | `role="combobox"`, `aria-autocomplete`, `aria-controls` |

---

## 10. Testing Checklist

Run these checks before marking any component or page as accessible.

### Automated

- [ ] Chrome DevTools → Lighthouse → Accessibility audit: score ≥ 90, zero critical violations
- [ ] Install [axe DevTools](https://www.deque.com/axe/) browser extension; run on every major page state (default, error, loading, empty)
- [ ] Verify all color contrast ratios with DevTools color picker or WebAIM Contrast Checker

### Colorblindness Simulation

- [ ] Chrome DevTools → Rendering panel → Emulate vision deficiencies:
  - [ ] Deuteranopia (green-blind) — can all status/states still be distinguished?
  - [ ] Protanopia (red-blind) — same check
  - [ ] Tritanopia (blue-blind) — verify blue accents remain distinguishable by shape/label
  - [ ] Achromatopsia (no color) — entire UI must be operable with zero color perception

### Keyboard Navigation

- [ ] Tab through the entire page without touching the mouse — every interactive element is reachable
- [ ] Focus indicator is always visible (bright blue ring or equivalent)
- [ ] Modal dialogs trap focus correctly and return focus on close
- [ ] Skip-to-content link appears on first Tab keypress

### Vision & Display

- [ ] Zoom browser to 200% — no horizontal scrolling, no content clipped or lost
- [ ] Zoom to 400% — content reflows to single column (WCAG 1.4.10 Reflow)
- [ ] Enable Windows High Contrast mode (Settings → Accessibility → Contrast themes) — UI remains usable
- [ ] Test on a non-retina display if possible — thin strokes and low-contrast elements are exposed

### Screen Reader (Spot Check)

- [ ] Activate NVDA (Windows) or VoiceOver (macOS/iOS) and navigate the page by headings — heading structure is logical
- [ ] All form inputs announce their label and any associated error message
- [ ] Dynamic content updates (toasts, errors, loading states) are announced

### Motion

- [ ] Enable "Reduce motion" in OS settings — verify no animations play (transitions collapse to instant)
- [ ] No content flashes more than 3 times per second in any state

---

## Quick Reference Card

```
Backgrounds:   #0f172a  #1e293b  #334155
Accents:       #3b82f6 (blue)  #f59e0b (amber)  #06b6d4 (cyan)  #a855f7 (purple)
Text:          #f8fafc (primary)  #94a3b8 (secondary)
Semantic:      #ef4444 (error)  #10b981 (success)  #f59e0b (warning)

Type:          16px min / 18px recommended / 1.6 line-height / 600 weight for labels
Spacing:       4 / 8 / 12 / 16 / 24 / 32 / 48 / 64 px
Targets:       44×44px touch / 32×32px click
Focus ring:    outline: 2px solid #3b82f6; outline-offset: 2px
Transition:    150ms ease (micro) / 250ms ease (panels)
Motion off:    prefers-reduced-motion: reduce → duration: 0.01ms

Rule:          Color + Icon + Text Label — always all three for semantic states
```
