# Code Review — `simple-html-table-builder.html`

**Scope:** last 5 commits (zebra striping refactor, bg-on-td, collapsible cards, UX cleanup, 3-column PA modal)
**Date:** 2026-06-13

---

## Findings

Ranked most-severe first. Lines are stable against the reviewed HEAD (`2c30909`).

---

### 1 · Select action name with apostrophe breaks the Compose expression

**File:** `simple-html-table-builder.html` · **Line:** 1135  
**Severity:** High — generated code is a PA syntax error; flow won't save

`selectNameSlug()` (line 1080) only replaces whitespace with underscores. It does not escape single-quotes. The slug is then interpolated raw into the `body('…')` call inside `composeExpr()`:

```js
return `concat(\n  ${paStr(open)},\n  join(body('${selectSlug}'), ''),\n  …\n)`;
```

If the user sets the Select action name to e.g. `"James's Select"`, the emitted expression becomes:

```
join(body('James's Select'), '')
```

— a PA expression syntax error (unescaped apostrophe terminates the string literal). Power Automate will refuse to save the Compose step.

**Fix:** escape the slug with the existing `paStr` helper before embedding, or call `.replace(/'/g, "''")` on it:

```js
join(body('${selectSlug.replace(/'/g, "''")}'), '')
```

---

### 2 · `loadState()` doesn't validate `rows` — crashes `refresh()` on first paint

**File:** `simple-html-table-builder.html` · **Line:** 878  
**Severity:** High — app is broken on page load; white screen with console error

The guard only checks for `columns`, `style`, and `pa`:

```js
if (p && Array.isArray(p.columns) && p.style && p.pa) return p;
```

A persisted state object that has those three keys but lacks a `rows` key (e.g. from an older app version, a schema migration, or a manually edited `localStorage` entry) passes the guard and is returned. On the next line, `refresh()` reads `state.rows.length` (line 1228) with no null-guard:

```js
if (!state.rows.length || !cols.length) {
```

→ `TypeError: Cannot read properties of undefined (reading 'length')`.

**Fix:** add `Array.isArray(p.rows)` to the validation condition, or default `state.rows = []` after the guard when the field is absent.

---

### 3 · Preview ignores `s.enabled` — styled preview for an unstyled PA table

**File:** `simple-html-table-builder.html` · **Line:** 1196  
**Severity:** Medium — preview and PA output look different when "Style the table" is unchecked

`rowMapExpr()` and `composeExpr()` both gate their style strings on `s.enabled`:

```js
// rowMapExpr – line 1096
const tdStyle = s.enabled ? `padding:…;${border}` : '';

// composeExpr – line 1123
const tableStyle = s.enabled ? `…font-family:…` : 'border-collapse:collapse;width:100%;';
```

`staticTableHtml()` — used for the live preview — never checks `s.enabled`. It always builds and applies `tableStyle`, `thStyle`, and `tdStyle` from the style values (lines 1202–1204). When the user unchecks "Style the table", the preview continues to show a fully styled table with header colours, padding, and borders, while the generated PA code produces a bare, unstyled table.

**Fix:** mirror the `s.enabled` guard in `staticTableHtml()`. When disabled, fall back to the same minimal style strings that `composeExpr` uses.

---

### 4 · `rule.value || ''` coerces numeric zero to empty string

**File:** `simple-html-table-builder.html` · **Line:** 1341  
**Severity:** Medium — numeric "equals 0" rules display blank and can silently break

```js
const valInput = el('input', {
  type: 'text', class: 'rule-val code', value: rule.value || '',
```

`0` is falsy, so the input renders blank when `rule.value === 0`. The `oninput` handler (line 1344) writes back `e.target.value` as a string, so if the user tabs through or edits the blank-looking field, `rule.value` is overwritten with the string `''` — silently breaking the rule (which now matches `equals ''` instead of `equals 0`). `buildColumnsPanel()` always recreates the input from state, so the blank display persists across rebuilds.

**Fix:** use nullish coalescing: `value: rule.value ?? ''`

---

### 5 · `parseTimer` can fire after `loadExampleData()`, stripping auto-colour

**File:** `simple-html-table-builder.html` · **Line:** 1517  
**Severity:** Medium — auto-coloured columns lose their rules silently

`loadExampleData()` doesn't clear the debounce timer:

```js
function loadExampleData() {
  $('jsonInput').value = EXAMPLE_JSON;
  state.columns = [];                   // ← auto-colour seeding intended here
  const res = ingest(state, EXAMPLE_JSON, true);   // autoColor = true
  buildColumnsPanel(); refresh();
}
```

If the user typed in the JSON textarea within the preceding 400 ms, `parseTimer` is still pending. After `loadExampleData` returns, the timer fires `parseInput()` → `ingest(state, EXAMPLE_JSON, false)` (autoColor = **false**). The second `ingest` call with `state.columns = []` having already been reset by the first call re-runs column detection without auto-colouring, erasing the colour rules just seeded.

**Fix:** add `clearTimeout(parseTimer);` as the first line of `loadExampleData()`.

---

### 6 · Collapse/expand toggle is never written to `localStorage`

**File:** `simple-html-table-builder.html` · **Line:** 1289  
**Severity:** Medium — collapse state is lost on every page reload

The card-head click handler mutates `col.collapsed` then calls only `buildColumnsPanel()`:

```js
head.addEventListener('click', e => {
  if (e.target.closest('input[type="checkbox"]')) return;
  col.collapsed = !col.collapsed;
  buildColumnsPanel();          // ← no refresh(), no persist()
});
```

`persist()` has a single call site: inside `refresh()` (line 1250). Since `buildColumnsPanel()` does not call `refresh()`, the collapse state change is never serialised to `localStorage`. The "Collapse all / Expand all" toolbar button shares the same problem (line 1554): it calls `buildColumnsPanel()` without `refresh()`.

**Fix:** call `persist()` directly after `buildColumnsPanel()` in both handlers (a full `refresh()` is not needed — only the state save).

---

### 7 · ∅ clear button disappears after re-picking colour via the picker

**File:** `simple-html-table-builder.html` · **Line:** 1385  
**Severity:** Low-Medium — user cannot clear a colour re-set via the picker without deleting the rule

The ∅ button is conditionally appended based on `rule.color` at render time:

```js
el('span', { class: 'swatch-label' }, 'text', colorBox, rule.color ? clearColor : null),
```

Clearing with ∅ sets `rule.color = ''` and calls `buildColumnsPanel()`, which rebuilds the row without the ∅ button (correct). If the user then picks a new colour via the `<input type="color">` picker, `oninput` fires `refresh()` but not `buildColumnsPanel()`. `rule.color` is now truthy, but the ∅ button is still absent from the DOM — the panel was not rebuilt. The user has no way to clear the colour without toggling collapse or performing any other action that triggers a panel rebuild.

**Fix:** the `oninput` handler on the colour picker should call `buildColumnsPanel()` in addition to `refresh()`, or the ∅ button should be toggled via a targeted DOM mutation rather than a full rebuild.

---

### 8 · Raw `pa.source` spliced into every field reference when zebra is on

**File:** `simple-html-table-builder.html` · **Line:** 1002  
**Severity:** Low — invalid source expression corrupts all column expressions at once; no validation or error

When zebra striping is enabled, `fieldRef()` embeds `state.pa.source` directly into every column's cell expression:

```js
const src = (state.pa.source || "outputs('Get_items')?['body/value']").trim();
return `${src}[item()]?['${String(key).replace(/'/g, "''")}']`;
```

A mistyped source (e.g. `outputs('Get_items')?[` — missing bracket) propagates into every `concat(…)` expression in the Select Map, producing expressions like:

```
outputs('Get_items')?[[item()]?['ColumnName']
```

All columns fail together, with no in-app warning. The hint in the modal shows the correct From expression separately, so the user has no visible signal that the source is also woven into the cell refs.

**Fix:** validate `pa.source` against a minimal pattern (non-empty, balanced brackets) before enabling zebra output, and show an inline error on the `paSource` field. Alternatively, restructure field refs to use `item()` for field access and keep the source expression confined to `zebraFromExpr()`.

---

## Summary table

| # | Line | Severity | Category | Status |
|---|------|----------|----------|--------|
| 1 | 1135 | High | Correctness | Fixed |
| 2 | 878 | High | Correctness | Fixed |
| 3 | 1196 | Medium | Correctness | Fixed |
| 4 | 1341 | Medium | Correctness | Fixed |
| 5 | 1517 | Medium | Correctness | Fixed |
| 6 | 1289 | Medium | Correctness | Fixed |
| 7 | 1385 | Low-Med | UX / Correctness | Fixed |
| 8 | 1002 | Low | Design / Robustness | Fixed |
