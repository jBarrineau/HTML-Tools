# Mono-Industrial Style Guide

A high-contrast, black-and-white visual style for single-page HTML tools. Heavy uppercase grotesque headings paired with monospace data text, thick black borders, and inverted (white-on-black) highlight pills. Built entirely from system fonts — **no external calls of any kind**.

---

## How to use this guide (instructions for the agent)

You are building a **single self-contained HTML file**. Follow these rules:

1. Copy the `:root` token block verbatim into a `<style>` tag. Derive every color, font, border, and radius from these tokens — do not hardcode ad-hoc values.
2. **Never** add `<link>` to fonts, `@import`, `@font-face`, CDN scripts, or any network request. Everything ships inline. This is non-negotiable (see Hard Rules).
3. Use the component patterns below as the building blocks. Match their structure and class names so the result is consistent.
4. Keep the palette monochrome. The one "color" move in this style is the inverted black pill — use it for emphasis, not decoration.
5. Author all heading and label text in normal sentence case in the HTML, and let `text-transform: uppercase` do the visual uppercasing. Do not type SHOUTING text into the markup.

---

## Aesthetic in one paragraph

Think operational, technical, and confident — a control-panel or dossier feel. The personality comes from the **type pairing** (a heavy grotesque shouting in uppercase next to plain monospace data) and from **structural weight** (thick `3px` ink borders, modest corner radius, generous padding). Color is almost absent: near-black ink on white paper, with solid-black pills inverting to white text for the one emphatic accent. Restraint is the point — let the two fonts and the heavy borders carry it.

---

## Hard rules (do not break)

- **No external resources.** No font links, no `@font-face`, no `@import url(...)`, no Google Fonts, no icon CDNs, no analytics, no remote images. Everything is inline in the single HTML file.
- **System fonts only.** Use the font stacks defined in the tokens. If a preferred face is absent on the user's OS, the stack falls back to another installed system face.
- **Monochrome by default.** Ink (`#111`) and paper (`#fff`) only. The optional accent token stays off unless explicitly requested.
- **Icons:** draw inline SVG (e.g. the heart/save glyph) — never load an icon font or sprite from the network.

---

## Design tokens

Paste this block into your `<style>`. These are the single source of truth.

```css
:root{
  /* ---- Color ---- */
  --ink:        #111111;   /* near-black: text, borders, pills */
  --ink-soft:   #4a4a4a;   /* secondary / muted text */
  --paper:      #ffffff;   /* cards and primary surfaces */
  --surface:    #f4f4f2;   /* page background so white cards read */
  --invert:     #ffffff;   /* text placed on an ink background */
  --line:       #111111;   /* full-strength borders (this style uses heavy ink lines) */
  --hairline:   rgba(17,17,17,.18); /* subtle dividers only */

  /* Optional single accent — OFF by default. Uncomment to introduce one color.
     --accent:  #c8102e;  */

  /* ---- Type (system fonts only, no external calls) ---- */
  /* Heavy grotesque for headings + labels. Arial Black is the reliable heavy face
     on Windows + macOS; alternatives that ship on Windows 11: "Bahnschrift"
     (condensed/industrial), "Impact" (very tight). */
  --font-display: "Arial Black", "Segoe UI", Arial, Helvetica, sans-serif;

  /* Monospace for data, values, tags, code. Consolas ships on Windows 11;
     the rest cover macOS / Linux without any download. */
  --font-mono: Consolas, "Cascadia Mono", ui-monospace, "SF Mono", Menlo, "Courier New", monospace;

  /* Neutral UI sans for longer body text / paragraphs inside tools. */
  --font-ui: system-ui, "Segoe UI", Roboto, Arial, sans-serif;

  /* ---- Structure ---- */
  --border-w:   3px;       /* signature thick card border */
  --radius-lg:  10px;      /* cards */
  --radius-sm:  4px;       /* pills, tags, inputs, buttons */

  /* ---- Spacing (use these, not magic numbers) ---- */
  --space-1: 8px;
  --space-2: 12px;
  --space-3: 16px;
  --space-4: 24px;
  --space-5: 34px;
}
```

### Why these font stacks
- **Display:** `Arial Black` gives the heavy, blocky uppercase punch on the two dominant desktop OSes. It runs wide, so headings get negative letter-spacing (see typography). On Windows 11 you may swap to `Bahnschrift` for a more condensed, industrial look in one line.
- **Mono:** `Consolas` is the Windows 11 default and a clean, neutral monospace. The fallbacks (`ui-monospace`, `SF Mono`, `Menlo`) keep macOS sharp and `Courier New` is the universal last resort.
- **UI sans:** `system-ui` resolves to the platform's native interface font (Segoe UI on Windows) for any running prose the tool needs.

---

## Typography

Two roles do almost all the work. Headings and labels use `--font-display` in uppercase; data and values use `--font-mono`. Reach for `--font-ui` only for genuine sentences/paragraphs.

| Role            | Font            | Size                       | Weight | Transform | Tracking | Line-height |
|-----------------|-----------------|----------------------------|--------|-----------|----------|-------------|
| Title (h1)      | `--font-display`| `clamp(22px, 3.2vw, 32px)` | 900    | uppercase | `-0.02em`| 1.1         |
| Section (h2)    | `--font-display`| 20px                       | 900    | uppercase | `-0.01em`| 1.15        |
| Label           | `--font-display`| 15px                       | 900    | uppercase | 0        | 1.3         |
| Value / data    | `--font-mono`   | 15px                       | 400    | none      | 0        | 1.3         |
| Tag / pill      | `--font-mono`   | 14px                       | 400    | none      | 0        | 1           |
| Body prose      | `--font-ui`     | 16px                       | 400    | none      | 0        | 1.6         |
| Caption / meta  | `--font-mono`   | 13px                       | 400    | none      | 0        | 1.4         |

```css
h1, .title{
  font-family: var(--font-display);
  font-weight: 900;
  text-transform: uppercase;
  letter-spacing: -0.02em;     /* Arial Black is wide — tighten it */
  line-height: 1.1;
  font-size: clamp(22px, 3.2vw, 32px);
  color: var(--ink);
  margin: 0;
}
.label{
  font-family: var(--font-display);
  font-weight: 900;
  text-transform: uppercase;
  color: var(--ink);
}
.value, .mono{ font-family: var(--font-mono); font-weight: 400; color: var(--ink); }
.prose{ font-family: var(--font-ui); font-size: 16px; line-height: 1.6; color: var(--ink); }
```

**Rule:** write the source text normally (`Category:`), uppercase it with CSS. This keeps content readable in the source, copy-paste friendly, and better for screen readers than literal capital letters.

---

## Color usage

This is a monochrome system. The discipline *is* the style.

- **Ink on paper** for nearly everything: `--ink` text on `--paper` surfaces.
- **Page background** is `--surface` (`#f4f4f2`) so white cards have an edge to sit against.
- **Inverted pill** is the single accent move: solid `--ink` background with `--invert` text, used for tags, status chips, and primary buttons. Use it sparingly — its power comes from being the only filled element.
- **Borders are full-strength ink**, not gray. Thickness (`--border-w`), not color, signals structure.
- **Optional accent:** if a project needs one color (e.g. a brand red for destructive actions or a single CTA), define `--accent` and apply it to exactly one element type. Do not sprinkle it.

Contrast is excellent by construction (`#111` on `#fff` ≈ 18:1), so accessibility targets are met as long as you keep text in `--ink` / `--invert`.

---

## Components

### Card

The core container: white surface, thick ink border, modest radius, roomy padding.

```html
<article class="card">
  <h1 class="title">Network engineer / communications systems</h1>
  <span class="pill">Camp Humphreys, Armed Forces Pacific</span>
  <div class="meta">
    <p class="row"><span class="label">Category:</span> <span class="value">Information Technology</span></p>
    <p class="row"><span class="label">Clearance:</span> <span class="value">Secret</span></p>
  </div>
</article>
```

```css
.card{
  background: var(--paper);
  border: var(--border-w) solid var(--line);
  border-radius: var(--radius-lg);
  padding: var(--space-5) 40px;
  max-width: 920px;
}
.meta{ display: flex; flex-direction: column; gap: var(--space-2); margin-top: var(--space-4); }
.row{ font-size: 15px; line-height: 1.3; margin: 0; }
.label{ margin-right: var(--space-1); }
```

### Inverted pill / tag

The signature accent. Solid ink, white monospace text.

```html
<span class="pill">Top Secret/SCI</span>
```

```css
.pill{
  display: inline-block;
  background: var(--ink);
  color: var(--invert);
  font-family: var(--font-mono);
  font-size: 14px;
  line-height: 1;
  padding: 9px 13px;
  border-radius: var(--radius-sm);
}
```

### Buttons

Primary mirrors the pill (filled ink). Secondary is outlined. Labels use the display font, uppercase.

```html
<button class="btn btn--primary">Run query</button>
<button class="btn btn--ghost">Reset</button>
```

```css
.btn{
  font-family: var(--font-display);
  font-weight: 900;
  text-transform: uppercase;
  letter-spacing: 0;
  font-size: 14px;
  padding: 10px 18px;
  border-radius: var(--radius-sm);
  border: var(--border-w) solid var(--ink);
  cursor: pointer;
  transition: transform .1s ease;
}
.btn:active{ transform: scale(.98); }
.btn:focus-visible{ outline: 2px solid var(--ink); outline-offset: 3px; }
.btn--primary{ background: var(--ink); color: var(--invert); }
.btn--ghost{ background: var(--paper); color: var(--ink); }
.btn--ghost:hover{ background: var(--surface); }
```

### Text input / select

White field, ink border, monospace value text so typed data matches the aesthetic.

```html
<label class="label" for="q">Search</label>
<input id="q" class="input" type="text" placeholder="keyword…">
```

```css
.input{
  display: block;
  width: 100%;
  font-family: var(--font-mono);
  font-size: 15px;
  color: var(--ink);
  background: var(--paper);
  border: 2px solid var(--ink);
  border-radius: var(--radius-sm);
  padding: 10px 12px;
  margin-top: var(--space-1);
}
.input::placeholder{ color: var(--ink-soft); }
.input:focus{ outline: 2px solid var(--ink); outline-offset: 2px; }
```

### Inline SVG icon (no icon font)

Draw glyphs inline so nothing loads externally. Example save/favorite heart:

```html
<button class="icon-btn" aria-label="Save" aria-pressed="false">
  <svg viewBox="0 0 24 24" width="28" height="28" fill="none" stroke="currentColor"
       stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
    <path d="M19.5 12.572 12 20l-7.5-7.428A5 5 0 1 1 12 6.006a5 5 0 1 1 7.5 6.566Z"/>
  </svg>
</button>
```

```css
.icon-btn{ background: none; border: 0; color: var(--ink); cursor: pointer; padding: 4px; border-radius: 6px; }
.icon-btn:focus-visible{ outline: 2px solid var(--ink); outline-offset: 3px; }
.icon-btn[aria-pressed="true"] svg{ fill: var(--ink); }
```

---

## Accessibility & quality floor

- Keep text in `--ink` or `--invert` — contrast is already strong; do not introduce low-contrast grays for primary content (`--ink-soft` is for secondary text only).
- Every interactive element gets a visible `:focus-visible` outline (`2px solid var(--ink)`, offset).
- Use real semantic elements (`button`, `label`, `input`, headings in order). Icon-only buttons need `aria-label`.
- Uppercase via CSS, not literal capitals, so screen readers and copy-paste stay clean.
- Respect motion preferences:

```css
@media (prefers-reduced-motion: reduce){
  *{ transition: none !important; animation: none !important; }
}
```

---

## Do / Don't

**Do**
- Lean on the two-font contrast (heavy display ↔ plain mono) as the primary identity.
- Keep borders thick and ink-colored; keep radii modest.
- Use the inverted pill as the single emphatic accent.
- Derive all values from the tokens.

**Don't**
- Don't load any external font, script, icon set, or image.
- Don't add gradients, drop shadows, glows, or multiple bright colors.
- Don't use thin gray borders — that erases the structural weight.
- Don't type ALL-CAPS into the HTML; transform with CSS.

---

## Starter template

A complete, self-contained single-page skeleton an agent can extend. No external calls.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Tool</title>
<style>
  :root{
    --ink:#111111; --ink-soft:#4a4a4a; --paper:#ffffff; --surface:#f4f4f2;
    --invert:#ffffff; --line:#111111; --hairline:rgba(17,17,17,.18);
    --font-display:"Arial Black","Segoe UI",Arial,Helvetica,sans-serif;
    --font-mono:Consolas,"Cascadia Mono",ui-monospace,"SF Mono",Menlo,"Courier New",monospace;
    --font-ui:system-ui,"Segoe UI",Roboto,Arial,sans-serif;
    --border-w:3px; --radius-lg:10px; --radius-sm:4px;
    --space-1:8px; --space-2:12px; --space-3:16px; --space-4:24px; --space-5:34px;
  }
  *{ box-sizing:border-box; }
  body{ margin:0; padding:40px 20px; background:var(--surface); color:var(--ink);
        font-family:var(--font-ui); -webkit-font-smoothing:antialiased;
        display:flex; justify-content:center; }
  .title{ font-family:var(--font-display); font-weight:900; text-transform:uppercase;
          letter-spacing:-0.02em; line-height:1.1; font-size:clamp(22px,3.2vw,32px); margin:0; }
  .label{ font-family:var(--font-display); font-weight:900; text-transform:uppercase; }
  .value{ font-family:var(--font-mono); }
  .card{ background:var(--paper); border:var(--border-w) solid var(--line);
         border-radius:var(--radius-lg); padding:var(--space-5) 40px; max-width:920px; width:100%; }
  .pill{ display:inline-block; background:var(--ink); color:var(--invert);
         font-family:var(--font-mono); font-size:14px; line-height:1;
         padding:9px 13px; border-radius:var(--radius-sm); margin:var(--space-4) 0; }
  .btn{ font-family:var(--font-display); font-weight:900; text-transform:uppercase; font-size:14px;
        padding:10px 18px; border-radius:var(--radius-sm); border:var(--border-w) solid var(--ink);
        cursor:pointer; }
  .btn--primary{ background:var(--ink); color:var(--invert); }
  .btn:focus-visible,.input:focus-visible{ outline:2px solid var(--ink); outline-offset:3px; }
  @media (prefers-reduced-motion: reduce){ *{ transition:none!important; animation:none!important; } }
</style>
</head>
<body>
  <main class="card">
    <h1 class="title">Tool name</h1>
    <span class="pill">Status: ready</span>
    <p class="value">Build your interface here using the tokens above.</p>
    <button class="btn btn--primary">Primary action</button>
  </main>
</body>
</html>
```
