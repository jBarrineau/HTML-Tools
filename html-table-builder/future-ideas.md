# HTML Table Builder — Future Ideas

Clever tricks and candidate features for combining Power Automate, HTML tables, and email.
Most exploit the same core insight as zebra striping and conditional colors: **anything you
can compute in an expression can be baked into the HTML per row.**

---

## Visual tricks inside the table

### 1. Data bars (mini bar charts)
Outlook can't do CSS charts, but it renders nested tables faithfully. Put a tiny two-cell
table inside a cell and drive the first cell's width from data:

```
width="@{div(mul(item()?['Amount'], 100), 5000)}%"
```

…with a colored `bgcolor`, and a gray second cell. You get an Excel-style data bar in every
row — pure HTML attributes, completely Outlook-safe.

### 2. Heatmap cells
Nested `if()`s mapping a value into color buckets (green / amber / red backgrounds).
Great for ageing reports — "days overdue" colored by severity.

```
@{if(greater(item()?['DaysOverdue'], 30), '#fde8e8', if(greater(item()?['DaysOverdue'], 7), '#fff7e6', '#e8f5e9'))}
```

### 3. Status badges with emoji
Emoji render reliably in every mail client and need zero styling:

```
@{if(equals(item()?['Status'],'Blocked'), '🔴', if(equals(item()?['Status'],'Done'), '✅', '🟡'))}
```

Cheaper than colored pills and survives dark mode, forwarding, and plain-text fallback.

### 4. Text-block sparklines
Power Automate has no string-repeat, but `substring()` over a constant fakes it:

```
substring('██████████', 0, <n>)
```

A five-block progress indicator per row with no images at all.

---

## Interactivity (the highest-payoff category)

### 5. Per-row action links
Add a **When an HTTP request is received** trigger to a second flow, then render a link
in each row:

```
<a href="https://...invoke?itemId=@{item()?['ID']}&action=approve">Approve</a>
```

Recipients click a link in the email and the flow updates the item — one-click approvals
straight from a table row.

> **Security:** the trigger URL contains a SAS token; treat it like a credential.
> Also validate the item's current state in the flow before acting on the request.

### 6. Deep links to the source item
SharePoint items expose a link field, so every row can jump to the real record:

```
<a href="@{item()?['{Link}']}">Open</a>
```

Same idea for Power Apps — open the app pre-navigated to a record:

```
https://apps.powerapps.com/play/<app-id>?ItemID=@{item()?['ID']}
```

---

## Data shaping before the table

### 7. Sorting and Top-N
Use expressions in the Select's **From**:

| Goal | Expression |
|---|---|
| Sort ascending | `sort(outputs('Get_items')?['body/value'], 'DueDate')` |
| Sort descending | `reverse(sort(…, 'DueDate'))` |
| Top 10 | `take(sort(…, 'DueDate'), 10)` |

### 8. Empty-state fallback
Nobody loves an empty table. Wrap the join:

```
@{if(equals(length(body('Select_table_rows')), 0),
  '<tr><td colspan="4" style="…">Nothing to report 🎉</td></tr>',
  join(body('Select_table_rows'), ''))}
```

Or better — skip sending the email entirely with a Condition on `length(…)`.

### 9. Totals row via the xpath trick
Power Automate has no `sum()`, but xpath does. Convert the array to XML, then sum:

```
xml(json(concat('{"root":{"r":', string(<array>), '}}')))
xpath(<that>, 'sum(//Amount)')
```

One expression, no Apply-to-each — render a bold footer row with the grand total.

### 10. Grouped sub-tables
`union()` over a Select of just the category field gives distinct group names; loop those
and run a **Filter array** + inner Select per group. More plumbing, but you get
"table per department" digests.

---

## Hygiene that bites people

- **Escape user content.** If a Title contains `<` or `&`, it corrupts the table. Chain
  `replace(replace(replace(x,'&','&amp;'),'<','&lt;'),'>','&gt;')` on free-text fields.
- **Multi-value fields** (people, choices): `join()` the array with `', '` or `'<br>'`
  rather than letting it stringify as JSON.
- **Outlook dark mode** inverts light backgrounds and can make white-on-dark headers
  unreadable. Mid-tone colors (like the Slate theme) survive inversion much better than
  pure white/black.

---

## Candidate builder features

Natural fits that extend the existing codegen:

- [ ] **Conditional formatting rules** — per-column rules ("when *field* equals/contains/greater-than
      *value*, set cell or row background/text color") with `if()` expressions generated
      automatically and the preview honoring rules against sample data
- [ ] **Data bar column type** — nested-table bars driven by a value and a max
- [ ] **Status badge / emoji column type** — value→emoji or value→color mappings
- [ ] **Link column type** — deep links to SharePoint items or Power Apps records
- [ ] **Empty-state option** — generated fallback row wrapped around the `join()`
- [ ] **Totals row option** — xpath `sum()` footer row
- [ ] **HTML-escape toggle** per text column — generated `replace()` chain
