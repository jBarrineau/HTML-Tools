# Pasting YAML Controls into Power Apps Canvas Studio (Browser Automation)

How an agent driving Power Apps Studio through the Chrome browser-automation
tools can reliably create a control by pasting its YAML into the canvas.

## The core problem

Power Apps Studio creates a control when you **paste control YAML** onto a
screen (`Ctrl+V`). The YAML must be on the **real OS clipboard**.

In the browser-automation context you **cannot** put text on the OS clipboard
the obvious way:

- `navigator.clipboard.writeText()` → throws `NotAllowedError`.
- `document.execCommand('copy')` → returns `false`.

Both fail because the automated tab does **not** hold OS-level focus
(`document.hasFocus()` returns `false`), and the async Clipboard API and
`execCommand` copy both require a focused document / user gesture.

## The key insight

Key events sent by the `computer` tool (via CDP) are **trusted, native**
events. A native `Ctrl+C` on a focused, selected `<textarea>` copies to the
**real OS clipboard** — the same path a human keypress takes. A native
`left_click` also grants the page genuine focus (after it,
`document.hasFocus()` becomes `true`).

So the winning pattern is: stage the YAML in a real on-page textarea, copy it
with native keys, then paste it onto the canvas with native keys.

## Procedure

### 1. Generate the control YAML

Match the structure of an existing control. Give it a **unique control name**,
and set a position that doesn't overlap siblings. Example:

```yaml
- Text2:
    Control: ModernText@1.0.0
    Properties:
      Fill: =RGBA(214, 221, 224, 1)
      Text: ="Goodbye, world!"
      Visible: =true
      X: =597
      Y: =400
```

Notes:
- The top-level key (`Text2`) is the control name — it must be unique on the screen.
- Property values are Power Fx expressions and start with `=`.
- Indentation is significant (2-space YAML). Keep it exact.

#### Discovering a control's type and properties

You usually won't know the exact `Control:` string (e.g. `ModernButton@1.0.0`)
or the real property names for a given control type. **Don't guess — copy an
existing control of that type and read its YAML back:**

1. Select one or more existing controls of the desired type. In the Tree view,
   `Ctrl+Click` to select several at once (e.g. a button *and* its label) — a
   single copy then yields all their YAML together, which is ideal when you want
   to reproduce a working interaction wholesale.
2. Native `Ctrl+C` (via the `computer` key tool) — Studio copies the selected
   control(s)' YAML to the OS clipboard.
3. Read it back by **pasting into an injected textarea** (see below) — do *not*
   rely on `navigator.clipboard.readText()`.

The YAML reveals the exact `Control:` type@version plus the property
names/values you can reuse. Note that exported YAML **omits properties left at
their default** (e.g. a button whose text is still the default has no `Text:`
line), so explicitly add any property you intend to set.

##### Reading the OS clipboard reliably (paste into a textarea)

`navigator.clipboard.readText()` is unreliable here: it needs the top document
focused, and after a `Ctrl+C` on a canvas/Tree-view control focus is stuck
inside the sandboxed iframe, so it throws `NotAllowedError` — and clicking a
top-level input does *not* dependably restore focus. Instead, reverse the
staging-textarea trick: paste the clipboard into a real textarea with native
keys, then read its `.value` (no clipboard permission, no focus required).

1. Inject a visible textarea (the snippet in step 2 below, but leave `value`
   empty).
2. Native `left_click` into it, then native `Ctrl+V`.
3. Read the value from the top page:

   ```js
   const ta = document.getElementById('cc_tmp_ta');
   "len=" + ta.value.length + "\n" + ta.value;
   ```

This is the same machinery steps 2–4 use for *writing* YAML, run in reverse,
and it works regardless of where focus currently sits.

### 2. Stage the YAML in a visible textarea (`javascript_tool`)

Create a real `<textarea>`, give it the YAML as its value, and make it visible
so a native click can land on it. Build the string with explicit `\n` newlines.

```js
const yaml = "- Text2:\n    Control: ModernText@1.0.0\n    Properties:\n      Fill: =RGBA(214, 221, 224, 1)\n      Text: =\"Goodbye, world!\"\n      Visible: =true\n      X: =597\n      Y: =400\n";
document.getElementById('cc_tmp_ta')?.remove();
const ta = document.createElement('textarea');
ta.id = 'cc_tmp_ta';
ta.value = yaml;
ta.style.cssText = 'position:fixed;top:150px;left:350px;width:600px;height:220px;z-index:2147483647;background:#fffbe6;border:3px solid red;font-size:14px;padding:8px;';
document.body.appendChild(ta);
// Return viewport info so you can map CSS px -> screenshot px.
"innerW=" + window.innerWidth + " dpr=" + window.devicePixelRatio;
```

### 3. Account for the screenshot ↔ CSS coordinate scale

Screenshots may be a different pixel size than the CSS viewport. Compute the
scale once and apply it to every coordinate you click:

```
scale = screenshotWidth / window.innerWidth
screenshotX = cssX * scale
screenshotY = cssY * scale
```

(Observed example: viewport `innerWidth = 1920`, screenshot width `1568`, so
`scale ≈ 0.817`. A textarea centered at CSS `(650, 260)` is at screenshot
`(~531, ~212)`.) When in doubt, take a screenshot and locate the element
(the red border makes it obvious) rather than trusting the math.

### 4. Copy with native keys (`computer` tool, ideally one `browser_batch`)

Click into the textarea (grants focus + user gesture), select all, copy:

```
left_click  -> center of the textarea
key         -> ctrl+a
key         -> ctrl+c
```

### 5. Verify the copy (optional)

The real proof the copy worked is a successful paste (step 8). If you want to
confirm the OS clipboard *before* pasting, paste into a scratch textarea and
read its `.value`, exactly as in "Reading the OS clipboard reliably" above —
don't use `navigator.clipboard.readText()`.

### 6. Remove the temporary textarea

```js
document.getElementById('cc_tmp_ta')?.remove();
```

### 7. Select the target screen, then paste onto the canvas

The control is pasted onto the **currently selected screen**, so first make sure
the right screen is active — click its node in the Tree view (the canvas switches
to it). Then give the **canvas screen** keyboard focus (not the Tree view):
clicking a Tree view node and pressing `Ctrl+V` does **not** paste.

To paste onto a **new** screen, create it first: Tree view → **New screen** →
**Blank**. Studio adds `ScreenN` and selects it automatically, so it's already
the active paste target — you can go straight to the canvas click + `Ctrl+V`.

```
left_click  -> the target screen's node in the Tree view   (selects the screen)
left_click  -> an empty area of the white screen in the canvas   (focus + target)
key         -> ctrl+v
```

The new control appears under that screen in the Tree view and renders on the
canvas at the `X`/`Y` from the YAML.

### 8. Validate the paste (critical — this is where double-pastes happen)

The paste is **asynchronous**. Studio takes a moment to commit the new control
and render it. If you check too early it looks like nothing happened, you
"retry," and you end up with duplicates (auto-renamed `Name_1`, `Name_2`, ...).

Rules that prevent that:

- **Never put `Ctrl+V` and the confirming `screenshot` in the same
  `browser_batch`.** The batch fires the screenshot milliseconds after the
  keypress, before the control exists → false negative → duplicate on retry.
  Paste in one step; verify in a *separate* step (add a short `wait` if needed).
- **The Tree view is the source of truth**, not the canvas render (the canvas
  can lag or the control may be off-screen / blank / behind another).
- **Use a count delta.** Before pasting, note the controls listed under the
  target screen. After pasting, confirm the count increased by *exactly* the
  number of controls in the YAML, and that the expected names are present.
- **Watch for the `_1` suffix.** An auto-renamed copy (`BtnGreet_1`) is the
  signature of an accidental double-paste. If you see one, you pasted twice —
  delete the extra (Tree view → right-click → **Delete**).
- **If a check looks empty, wait and re-check — do NOT re-paste.** Re-pasting
  without first re-reading the tree is what creates the duplicate.

> You cannot script-read the tree. The entire Studio designer — canvas **and**
> Tree view — renders inside a sandboxed iframe whose `contentDocument` is
> `null`, so `document.querySelector` from the top page finds none of it (it
> only sees the temporary textareas you inject). Validation is therefore a
> **screenshot of the Tree view**, read as a properly-timed separate step.

## Wiring interactions and verifying behavior

Controls interact through variables, and that wiring can live entirely in the
pasted YAML. The common "button updates a label" pattern:

- Button: `OnSelect: =Set(varGreet, "Hello from Chrome!")`
- Label:  `Text: =varGreet`   (blank until the button sets the variable)

You can create both at once — put several `- Name:` list items in one YAML
block and paste it in a single operation (then validate the count delta for
*all* of them per step 8).

**Verify behavior in Preview**, because pasted logic (`OnSelect`, bindings)
can't be exercised reliably from edit mode (the `Alt+Left-Click` run gesture
isn't reproducible through the automation tools — see Gotchas):

1. Select the screen you want to test — Preview starts from the current screen.
2. Click the **▶ Play** button (not `F5` — see Gotchas).
3. Interact (e.g. click the button) and confirm the outcome (e.g. the label now
   shows the expected text).
4. Exit Preview with the **Edit** button.

Note: variables `Set` during Preview persist for the rest of the editing
session, so the label may still show its post-click value back in edit mode —
that is expected session state, not a stray/duplicate control.

## Gotchas / lessons learned

- **Don't rely on `navigator.clipboard` / `execCommand` to write** — they fail
  without OS focus. Route the copy through native `Ctrl+C` on a real element.
- **Native `left_click` restores focus.** That's why clipboard reads work after
  step 4 but not before.
- **Paste target matters.** Tree view node selected + `Ctrl+V` = nothing.
  Click the screen surface in the canvas first.
- **Screenshot pixels ≠ CSS pixels.** Always convert with the scale factor.
- **`javascript_tool` returns the last expression.** Prefer top-level `await`;
  avoid returning a bare Promise (you'll see `{}`).
- **Paste is async → verify in a separate, timed step.** Bundling `Ctrl+V` with
  the confirming screenshot causes false negatives and duplicate pastes. See
  step 8; check the Tree view with a count delta and watch for `_1` suffixes.
- **Editing a control in the canvas requires `Alt+Left-Click`**, and the
  automation tools apply the Alt modifier only for the click instant — they
  can't *hold* Alt with a real gap before mousedown, so running a control's
  `OnSelect` from edit mode this way is unreliable. To actually trigger a
  control's behavior, use **Preview mode** and click normally, then exit with
  the **Edit** button.
- **`F5` does not launch Preview through the automation tool.** The `computer`
  key tool maps `F5` to the *browser's* page-reload command, so it reloads
  Studio instead of dispatching an F5 keystroke to the page. Launch Preview by
  clicking the **▶ Play** button (it previews from the currently selected
  screen), not with `F5`.
