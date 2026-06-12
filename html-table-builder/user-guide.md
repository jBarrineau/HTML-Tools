# HTML Table Builder — User Guide

Design Outlook-safe HTML tables in your browser, then drop the generated pieces into a
Power Automate flow so the **Send an email** action renders your live data as a styled table.

Everything runs locally — no data leaves the page, and your design autosaves to the
browser's localStorage between visits.

---

## 1. The big picture

Power Automate's built-in **Create HTML table** action produces an unstyled table that looks
poor in Outlook, and Outlook desktop ignores `<style>` blocks and CSS classes entirely
(it renders email with the Word engine). The only reliable way to style email tables is to
inline every style on every tag — tedious by hand.

This tool generates that HTML for you, in two halves:

1. **A row template** — one `<tr>` with `@{…}` expression tokens where your data goes.
   You paste this into a **Select** action's Map, which stamps out one row string per item
   in your array.
2. **A table wrapper** — the `<table>`, header row, and a `join()` expression that stitches
   the Select's output rows into the body. You paste this into the email body's code view.

At run time: your array → Select produces `["<tr>…</tr>", "<tr>…</tr>", …]` →
`join(…, '')` concatenates them → the email contains a complete, fully styled table.

---

## 2. Tour of the designer

The left panel has four tabs; the right side shows a live preview (styled as an Outlook
message) and the generated output.

### Columns tab

Each card is one table column, rendered left-to-right in card order.

| Setting | What it does |
|---|---|
| **Header** | The text shown in the table's header row. |
| **Field / =expression** | The property name in your array items — becomes `item()?['Field']`. Start with `=` to supply a raw expression instead (see below). |
| **Format** | Text, Number, Currency, or Date. Number/Currency/Date wrap the value in `formatNumber()` / `formatDateTime()`. |
| **Format string** | The .NET format string, e.g. `yyyy-MM-dd` or `MMM dd` for dates, `C2` for currency, `N0` for whole numbers with separators. |
| **Align** | Auto aligns text left and numbers/currency right; or force left/center/right. |
| **Width** | Pixels (`120`) or percent (`25%`). Leave blank for automatic. |
| **Bold / No wrap / Text color** | Per-column cell styling. *No wrap* keeps dates and amounts on one line. |

Use **↑ ↓** to reorder columns and **✕** to remove one. Sample data moves with the column.

**Raw expressions.** Prefix the Field with `=` for anything beyond a simple property:

- Nested property: `=item()?['Author']?['DisplayName']`
- Combined fields: `=concat(item()?['FirstName'], ' ', item()?['LastName'])`
- Calculation: `=mul(item()?['Qty'], item()?['UnitPrice'])`

### Style tab

- **Theme presets** — six Outlook-tested palettes (Slate, Corporate Blue, Forest, Burgundy,
  Graphite, Minimal). A preset sets header, row, border, band, and accent colors; everything
  remains individually adjustable afterward.
- **Table** — width (fluid 100% or fixed pixels), font, base size, text color, row
  background, and an optional 4 px accent bar across the top.
- **Header row** — background, text color, size, bold, UPPERCASE. (Uppercase is baked into
  the generated text because Outlook ignores `text-transform`.)
- **Borders & rows** — horizontal rules, full grid, outer frame, or none, plus border color.
- **Zebra striping** — banded rows. ⚠ Turning this on changes the generated Power Automate
  pattern; see [section 4](#4-variant-zebra-striping-the-indexed-pattern).
- **Cell padding** — vertical and horizontal padding inside every cell.

### Sample Data tab

Editable rows that drive the live preview and the Static HTML output. They are stand-ins
only — at run time Power Automate supplies the real rows. Enter dates as `2026-06-30`-style
strings and numbers without currency symbols so the format preview works.

### Flow tab

| Setting | What it does |
|---|---|
| **Source array** | The expression that returns your array of items. This appears in the generated **From** field. |
| **Select action name** | Must match the name of your Select action in the flow — it's used in the `join(body('…'))` expression. |
| **Null-safe formatting** | Wraps `formatDateTime()` / `formatNumber()` in `if(empty(…), '', …)` so a blank field renders as an empty cell instead of failing the flow. Leave this on. |

Common source arrays:

| Data source | Expression |
|---|---|
| SharePoint **Get items** | `outputs('Get_items')?['body/value']` |
| Dataverse **List rows** | `outputs('List_rows')?['body/value']` |
| A Filter array action | `body('Filter_array')` |
| An array variable | `variables('myArray')` |

> Action names with spaces become underscores inside expressions: an action named
> "Get items" is referenced as `outputs('Get_items')`.

---

## 3. Setting up the Power Automate flow (step by step)

This walkthrough uses SharePoint **Get items**, but any action that returns an array works
the same way. The output panel's **Power Automate setup** tab gives you the three pieces
in order, each with a Copy button.

### Step 0 — Get your data

Add the action that returns your items, e.g. **Get items** (SharePoint). Note its exact
name — if you renamed it to "Get overdue tasks", your source array expression must be
`outputs('Get_overdue_tasks')?['body/value']`. Update the **Flow tab → Source array**
to match, and the generated code updates instantly.

> **SharePoint tip:** field names in expressions are *internal* names, not display names.
> A column displayed as "Due Date" is often internally `DueDate` or `Due_x0020_Date`.
> Check the column's internal name in list settings (it's in the URL), or run the flow once
> and inspect the Get items raw outputs.

### Step 1 — Add a Select action and set From

1. Below your data action, add a **Select** action (under *Data Operation*).
2. Rename it to match the **Flow tab → Select action name** (default: `Select table rows`).
   The name and the `join()` in step 3 must agree.
3. Click into the **From** field, open the expression editor (the *fx* button), and paste
   the **Step 1** output from the tool. For the standard pattern this is just your source
   array, e.g.:

   ```
   outputs('Get_items')?['body/value']
   ```

### Step 2 — Switch Map to text mode and paste the row template

This is the step people miss. By default, Select's **Map** is a key/value grid that
produces JSON objects — you need it to produce plain strings instead.

1. On the right-hand edge of the **Map** field, find the small toggle button
   (it looks like **🗗** / "Switch Map to text mode" — in the new designer it's the icon
   to the right of the Map label).
2. Click it. The key/value grid collapses into a single text box.
3. Paste the **Step 2** output (the row template) into that box, exactly as copied.
   It looks like:

   ```html
   <tr bgcolor="#ffffff" style="background-color:#ffffff;"><td align="left" style="padding:8px 12px;…">@{item()?['Title']}</td>…</tr>
   ```

The `@{…}` tokens are live expressions — Power Automate evaluates them per item. After
pasting, the dynamic-content tokens may render as pills; that's fine.

> **Paste as plain text.** If the designer mangles the paste, clear the field and paste
> again with `Ctrl+Shift+V`. Don't retype the template by hand.

### Step 3 — Paste the table into Send an email

1. Add **Send an email (V2)** (Office 365 Outlook).
2. Fill in To and Subject as usual.
3. In the **Body** field's toolbar, click the **`</>`** button (code view). This is
   essential — pasting into the normal rich-text view HTML-encodes your markup and the
   email will show raw tags.
4. Paste the **Step 3** output. It contains your styled `<table>`, the header row, and:

   ```
   @{join(body('Select_table_rows'), '')}
   ```

   inside `<tbody>` — that expression injects all the rows the Select produced.

You can surround the table with your own greeting and signature text in the same code
view (`<p>Hi team,</p>` above, etc.).

### Step 4 — Test

Run the flow. Check:

- The Select action's **OUTPUT** should be an array of `<tr>…</tr>` strings.
- The received email should show the styled table.

If something is off, see [Troubleshooting](#6-troubleshooting).

---

## 4. Variant: zebra striping (the indexed pattern)

A Select map normally can't know which row number it's on, so alternating row colors are
impossible with the simple `item()` pattern. When you enable **Zebra striping** on the
Style tab, the tool switches to an index-based pattern automatically:

- **From** becomes a range over the row numbers:

  ```
  range(0, length(outputs('Get_items')?['body/value']))
  ```

- In the **Map**, `item()` is now the row *index*, so field references index into your
  array — `outputs('Get_items')?['body/value'][item()]?['Title']` — and the row color is
  computed from the index:

  ```
  @{if(equals(mod(item(), 2), 0), '#ffffff', '#f1f5f9')}
  ```

Nothing changes in your flow steps — paste From and Map the same way. The only caveat:
if a column uses a raw `=` expression, make sure it doesn't assume `item()` is the data
item, because in this mode it's the index.

---

## 5. Output reference

| Output | Where it goes |
|---|---|
| **Step 1 — From** | Select action → From field (via the expression editor or pasted directly). |
| **Step 2 — Map (text mode)** | Select action → Map field, after switching to text mode. |
| **Step 3 — Email body** | Send an email (V2) → Body, in code view (`</>`). |
| **Static HTML** tab | The complete table with your sample data baked in. Use it to send yourself a test email before wiring up the flow, or as a hand-edited static table. |

**Import / Export / Reset** (top right) save and load the whole design as a JSON file —
useful for sharing a house style with colleagues or keeping per-report designs.

---

## 6. Troubleshooting

**The email shows raw HTML tags.**
You pasted into the rich-text body instead of code view. Click the `</>` button in the
body toolbar first, then paste.

**Cells are empty but rows appear.**
The field name doesn't match. SharePoint expressions need *internal* column names
(`Due_x0020_Date`, not "Due Date"). Run the flow and inspect the Get items raw outputs to
see the real property names.

**The flow fails on `formatDateTime` / `formatNumber` with "expected … got Null".**
A row has a blank value. Turn on **Null-safe formatting** in the Flow tab and re-copy the
Map template.

**The flow fails with "the template language function 'join' …".**
The action name in `join(body('Select_table_rows'), '')` doesn't match your Select action's
actual name. Spaces become underscores; renaming the action after pasting breaks the
reference. Fix the name in the Flow tab and re-copy the Step 3 output.

**The Map only lets me enter key/value pairs.**
You're still in key-value mode. Click the small toggle at the right edge of the Map field
to switch to text mode (step 2 above).

**Dates show as `2026-06-30T00:00:00Z`.**
The column's Format is set to Text. Change it to Date and pick a format string.

**The table looks different in Outlook desktop vs. the preview.**
Minor differences are normal — Outlook's Word engine approximates some spacing. The
generated HTML already avoids the big traps (no `<style>` block, no classes, no
`text-transform`, legacy `bgcolor`/`align`/`width` attributes included). If colors look
inverted, the recipient is in Outlook dark mode; mid-tone themes like Slate survive
inversion best.

**Zebra striping shows the same color on every row.**
You enabled zebra *after* pasting, so the flow still has the old simple-pattern From/Map.
Re-copy **both** Step 1 and Step 2 — the indexed pattern needs them as a matched pair.

---

## 7. Worked example

Goal: a weekly email of overdue tasks from a SharePoint list "Tasks" with columns
Title, Status, DueDate, Amount.

1. **Flow trigger:** Recurrence, weekly.
2. **Get items** on the Tasks list, with an OData filter like
   `DueDate lt '@{utcNow()}' and Status ne 'Completed'`.
3. **In the tool:** keep the four default columns (they match this schema), pick the
   Slate theme, set Flow tab → Source array to `outputs('Get_items')?['body/value']`.
4. **Select** action: paste From (Step 1) and Map in text mode (Step 2).
5. **Send an email (V2):** body in code view, paste Step 3, add a greeting above the table.
6. Run it. Four overdue tasks become four styled rows; zero tasks produce an empty table —
   consider adding a Condition on `length(outputs('Get_items')?['body/value'])` so the
   email only sends when there's something to report.
