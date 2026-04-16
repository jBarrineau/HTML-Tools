# Canvas Apps YAML Patterns Reference

This file documents the exact YAML syntax used in Power Apps Studio for responsive layouts. The code you produce can be copy-pasted directly into Canvas Apps Studio using the "Paste code" feature (right-click on a screen or container in the tree view → "Paste code").

## How YAML code is used in Canvas Apps

When pasting controls into Canvas Apps Studio, you paste only the **control subtree** — not the `Screens:` wrapper. The format is a YAML list of control objects. Each control is a named item with a `Control:` type, optional `Variant:`, optional `Properties:`, and optional `Children:`.

All property values that use formulas or expressions must be prefixed with `=`.  
Literal strings (like color names, enum values) also use `=` when set as formulas.

---

## CORRECT CONTROL VERSIONS (verified against live Power Apps Studio)

| Control | Correct version |
|---------|----------------|
| GroupContainer | `GroupContainer@1.5.0` |
| Rectangle | `Rectangle@2.3.0` |
| Circle | `Circle@2.3.0` |

The variant for auto-layout containers is `AutoLayout` (not `verticalAutoLayoutContainer` / `horizontalAutoLayoutContainer`).  
Direction is set via the `LayoutDirection` property:
- Vertical stack: `LayoutDirection: =LayoutDirection.Vertical`
- Horizontal stack: `LayoutDirection: =LayoutDirection.Horizontal`

Manual layout variant is `ManualLayout`.

---

## Screen-level root container pattern

Outermost container that fills the full screen (vertical stack):

```yaml
- RootContainer:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      X: =0
      Y: =0
      Width: =Parent.Width
      Height: =Parent.Height
      LayoutDirection: =LayoutDirection.Vertical
      LayoutAlignItems: =LayoutAlignItems.Stretch
      LayoutJustifyContent: =LayoutJustifyContent.Start
      Fill: =RGBA(0,0,0,0)
      PaddingTop: =0
      PaddingBottom: =0
      PaddingLeft: =0
      PaddingRight: =0
```

---

## Horizontal Container

```yaml
- HorizontalContainer1:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      AlignInContainer: =AlignInContainer.Stretch
      FillPortions: =1
      LayoutDirection: =LayoutDirection.Horizontal
      LayoutAlignItems: =LayoutAlignItems.Stretch
      LayoutJustifyContent: =LayoutJustifyContent.Start
      LayoutGap: =0
      LayoutOverflowX: =LayoutOverflow.Hide
      LayoutOverflowY: =LayoutOverflow.Hide
      LayoutWrap: =false
      Fill: =RGBA(0,0,0,0)
      PaddingTop: =0
      PaddingBottom: =0
      PaddingLeft: =0
      PaddingRight: =0
    Children: []
```

**Key property values for `LayoutJustifyContent` (primary axis = horizontal):**
- `=LayoutJustifyContent.Start`
- `=LayoutJustifyContent.End`
- `=LayoutJustifyContent.Center`
- `=LayoutJustifyContent.SpaceBetween`

**Key property values for `LayoutAlignItems` (cross axis = vertical):**
- `=LayoutAlignItems.Start`
- `=LayoutAlignItems.End`
- `=LayoutAlignItems.Center`
- `=LayoutAlignItems.Stretch`

---

## Vertical Container

```yaml
- VerticalContainer1:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      AlignInContainer: =AlignInContainer.Stretch
      FillPortions: =1
      LayoutDirection: =LayoutDirection.Vertical
      LayoutAlignItems: =LayoutAlignItems.Stretch
      LayoutJustifyContent: =LayoutJustifyContent.Start
      LayoutGap: =0
      LayoutOverflowX: =LayoutOverflow.Hide
      LayoutOverflowY: =LayoutOverflow.Hide
      LayoutWrap: =false
      Fill: =RGBA(0,0,0,0)
      PaddingTop: =0
      PaddingBottom: =0
      PaddingLeft: =0
      PaddingRight: =0
    Children: []
```

---

## Regular (Manual Layout) Container

```yaml
- Container1:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      X: =0
      Y: =0
      Width: =Parent.Width
      Height: =Parent.Height
      Fill: =RGBA(0,0,0,0)
```

---

## Rectangle Shape

> ⚠️ `Rectangle@2.3.0` does NOT support `RadiusTopLeft`, `RadiusTopRight`, `RadiusBottomLeft`, `RadiusBottomRight`. Rectangles are always sharp-cornered. To get a rounded rectangle, use a `GroupContainer@1.5.0` with `Variant: ManualLayout` and set its `Fill` and `Radius*` properties instead.

```yaml
- Rectangle1:
    Control: Rectangle@2.3.0
    Properties:
      Fill: =RGBA(56, 96, 178, 1)
      Width: =Parent.Width
      Height: =Parent.Height
      X: =0
      Y: =0
```

## Rounded rectangle (use GroupContainer instead of Rectangle)

```yaml
- RoundedCard:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      Fill: =RGBA(255, 255, 255, 1)
      Width: =Parent.Width
      Height: =120
      RadiusTopLeft: =8
      RadiusTopRight: =8
      RadiusBottomLeft: =8
      RadiusBottomRight: =8
```

## Circle Shape

```yaml
- Circle1:
    Control: Circle@2.3.0
    Properties:
      Fill: =RGBA(56, 96, 178, 1)
      Width: =60
      Height: =60
      X: =0
      Y: =0
```

---

## `AlignInContainer` (child property — how a child aligns inside its parent auto-layout container)

```yaml
AlignInContainer: =AlignInContainer.SetByContainer  # inherits from parent
AlignInContainer: =AlignInContainer.Start
AlignInContainer: =AlignInContainer.Center
AlignInContainer: =AlignInContainer.End
AlignInContainer: =AlignInContainer.Stretch
```

## `FillPortions` (child property — proportional sizing along primary axis)

```yaml
FillPortions: =1   # takes equal share of remaining space
FillPortions: =2   # takes double the share of a sibling with FillPortions=1
FillPortions: =0   # does not grow (fixed size; set explicit Width or Height instead)
```

When `FillPortions` is used, the container distributes free space proportionally. A child with `FillPortions=0` is fixed-size; others grow to fill.

---

## Common Responsive Layout Patterns

### Full-screen root (place directly on a screen)

```yaml
- ScreenRoot:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
      LayoutDirection: =LayoutDirection.Vertical
    Properties:
      X: =0
      Y: =0
      Width: =Parent.Width
      Height: =Parent.Height
      LayoutAlignItems: =LayoutAlignItems.Stretch
      Fill: =RGBA(255,255,255,1)
```

### Fixed-height header + scrollable content body

```yaml
- AppShell:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
      LayoutDirection: =LayoutDirection.Vertical
    Properties:
      X: =0
      Y: =0
      Width: =Parent.Width
      Height: =Parent.Height
      LayoutAlignItems: =LayoutAlignItems.Stretch
      LayoutJustifyContent: =LayoutJustifyContent.Start
      Fill: =RGBA(255,255,255,1)
    Children:
      - Header:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
      LayoutDirection: =LayoutDirection.Horizontal
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =0
            Height: =64
            LayoutAlignItems: =LayoutAlignItems.Center
            LayoutJustifyContent: =LayoutJustifyContent.SpaceBetween
            Fill: =RGBA(56, 96, 178, 1)
            PaddingLeft: =16
            PaddingRight: =16
      - Body:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
      LayoutDirection: =LayoutDirection.Vertical
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =1
            LayoutAlignItems: =LayoutAlignItems.Stretch
            LayoutOverflowY: =LayoutOverflow.Scroll
            Fill: =RGBA(0,0,0,0)
```

### Sidebar (left nav) + main content

```yaml
- AppLayout:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
      LayoutDirection: =LayoutDirection.Horizontal
    Properties:
      X: =0
      Y: =0
      Width: =Parent.Width
      Height: =Parent.Height
      LayoutAlignItems: =LayoutAlignItems.Stretch
      Fill: =RGBA(255,255,255,1)
    Children:
      - Sidebar:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
      LayoutDirection: =LayoutDirection.Vertical
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =0
            Width: =240
            LayoutAlignItems: =LayoutAlignItems.Stretch
            LayoutJustifyContent: =LayoutJustifyContent.Start
            Fill: =RGBA(33, 37, 41, 1)
      - MainContent:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
      LayoutDirection: =LayoutDirection.Vertical
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =1
            LayoutAlignItems: =LayoutAlignItems.Stretch
            LayoutOverflowY: =LayoutOverflow.Scroll
            Fill: =RGBA(248,249,250,1)
```

### Split-screen (50/50)

```yaml
- SplitScreen:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
      LayoutDirection: =LayoutDirection.Horizontal
    Properties:
      X: =0
      Y: =0
      Width: =Parent.Width
      Height: =Parent.Height
      LayoutAlignItems: =LayoutAlignItems.Stretch
      Fill: =RGBA(255,255,255,1)
    Children:
      - LeftPanel:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
      LayoutDirection: =LayoutDirection.Vertical
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =1
            LayoutAlignItems: =LayoutAlignItems.Stretch
            Fill: =RGBA(240,240,240,1)
      - RightPanel:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
      LayoutDirection: =LayoutDirection.Vertical
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =1
            LayoutAlignItems: =LayoutAlignItems.Stretch
            Fill: =RGBA(255,255,255,1)
```

### Card grid row (N equal-width cards in a horizontal row)

```yaml
- CardRow:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
      LayoutDirection: =LayoutDirection.Horizontal
    Properties:
      AlignInContainer: =AlignInContainer.Stretch
      FillPortions: =0
      Height: =200
      LayoutAlignItems: =LayoutAlignItems.Stretch
      LayoutJustifyContent: =LayoutJustifyContent.Start
      LayoutGap: =16
      PaddingLeft: =16
      PaddingRight: =16
      Fill: =RGBA(0,0,0,0)
    Children:
      - Card1:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
      LayoutDirection: =LayoutDirection.Vertical
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =1
            LayoutAlignItems: =LayoutAlignItems.Center
            LayoutJustifyContent: =LayoutJustifyContent.Center
            Fill: =RGBA(255,255,255,1)
            RadiusTopLeft: =8
            RadiusTopRight: =8
            RadiusBottomLeft: =8
            RadiusBottomRight: =8
      - Card2:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
      LayoutDirection: =LayoutDirection.Vertical
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =1
            LayoutAlignItems: =LayoutAlignItems.Center
            LayoutJustifyContent: =LayoutJustifyContent.Center
            Fill: =RGBA(255,255,255,1)
            RadiusTopLeft: =8
            RadiusTopRight: =8
            RadiusBottomLeft: =8
            RadiusBottomRight: =8
```

---

## Color formulas

Use `RGBA(r, g, b, a)` for colors. Some common values:
- Transparent: `=RGBA(0,0,0,0)`
- White: `=RGBA(255,255,255,1)`
- Light gray: `=RGBA(248,249,250,1)`
- Medium gray: `=RGBA(200,200,200,1)`
- Dark nav: `=RGBA(33,37,41,1)`
- Blue accent: `=RGBA(56,96,178,1)`

---

## Important rules

1. **All properties use `=` prefix** — even non-formula values like `=0` or `=true`.
2. **Do NOT set X, Y inside auto-layout containers** — the container manages position for its children. Only set X and Y on controls placed directly on a screen or inside a manual-layout container.
3. **FillPortions=0 means fixed size** — pair it with an explicit `Width` or `Height`.
4. **Root container always uses `Width: =Parent.Width` and `Height: =Parent.Height`** so it stretches with the screen.
5. **Overflow + Scroll** — to make a section scrollable, set `LayoutOverflowY: =LayoutOverflow.Scroll` and ensure its parent is giving it a bounded height (e.g., via Stretch or fixed height).
6. **Shapes inside auto-layout containers** — do NOT set X/Y. Set Width/Height to fixed values or use FillPortions for proportional sizing.
7. **`Parent.Width` formula in ManualLayout children** — `Width: =Parent.Width * 0.7` works inside a `ManualLayout` container, but be aware the ManualLayout child inside an AutoLayout track may not always resolve the parent correctly at very small sizes (e.g., 4px progress tracks). For reliable progress bars, consider making the track taller (≥8px) or using two sibling containers with different `FillPortions` values instead.
8. **Bar charts with bottom-aligned columns** — use `LayoutAlignItems: =LayoutAlignItems.End` + `LayoutJustifyContent: =LayoutJustifyContent.SpaceBetween` on the columns container, with `AlignInContainer: =AlignInContainer.End` on each bar. Each bar should use `Variant: ManualLayout` with explicit `Height` and top-only radius. Verified working in live Power Apps Studio.

---

## Validated complex patterns (verified in Power Apps Studio 2026-04-08)

The following patterns were confirmed working in a full analytics dashboard layout:

### Bottom-aligned bar chart
```yaml
- ChartColumns:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      AlignInContainer: =AlignInContainer.Stretch
      FillPortions: =1
      LayoutDirection: =LayoutDirection.Horizontal
      LayoutAlignItems: =LayoutAlignItems.End
      LayoutJustifyContent: =LayoutJustifyContent.SpaceBetween
      LayoutGap: =8
      Fill: =RGBA(0,0,0,0)
      PaddingTop: =0
      PaddingBottom: =0
      PaddingLeft: =0
      PaddingRight: =0
    Children:
      - Bar1:
          Control: GroupContainer@1.5.0
          Variant: ManualLayout
          Properties:
            AlignInContainer: =AlignInContainer.End
            FillPortions: =1
            Height: =120
            Fill: =RGBA(56,96,178,1)
            RadiusTopLeft: =3
            RadiusTopRight: =3
            RadiusBottomLeft: =0
            RadiusBottomRight: =0
```

### Pill/badge (fully rounded rectangle)
```yaml
- StatusBadge:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      AlignInContainer: =AlignInContainer.Center
      FillPortions: =0
      Width: =64
      Height: =24
      Fill: =RGBA(220,252,231,1)
      RadiusTopLeft: =12
      RadiusTopRight: =12
      RadiusBottomLeft: =12
      RadiusBottomRight: =12
```

### Row with avatar + text + badge (table row pattern)
```yaml
- TableRow:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      AlignInContainer: =AlignInContainer.Stretch
      FillPortions: =0
      Height: =48
      LayoutDirection: =LayoutDirection.Horizontal
      LayoutAlignItems: =LayoutAlignItems.Center
      LayoutGap: =0
      Fill: =RGBA(255,255,255,1)
      PaddingLeft: =20
      PaddingRight: =20
      PaddingTop: =0
      PaddingBottom: =0
    Children:
      - NameCell:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
          Properties:
            AlignInContainer: =AlignInContainer.Center
            FillPortions: =2
            LayoutDirection: =LayoutDirection.Horizontal
            LayoutAlignItems: =LayoutAlignItems.Center
            LayoutGap: =10
            Fill: =RGBA(0,0,0,0)
            PaddingTop: =0
            PaddingBottom: =0
            PaddingLeft: =0
            PaddingRight: =0
          Children:
            - Avatar:
                Control: Circle@2.3.0
                Properties:
                  FillPortions: =0
                  Width: =28
                  Height: =28
                  Fill: =RGBA(56,96,178,0.2)
            - NameBar:
                Control: Rectangle@2.3.0
                Properties:
                  FillPortions: =0
                  Width: =100
                  Height: =14
                  Fill: =RGBA(30,41,59,1)
      - DataCol:
          Control: Rectangle@2.3.0
          Properties:
            AlignInContainer: =AlignInContainer.Center
            FillPortions: =1
            Height: =12
            Fill: =RGBA(100,116,139,1)
      - BadgeCol:
          Control: GroupContainer@1.5.0
          Variant: ManualLayout
          Properties:
            AlignInContainer: =AlignInContainer.Center
            FillPortions: =0
            Width: =64
            Height: =24
            Fill: =RGBA(220,252,231,1)
            RadiusTopLeft: =12
            RadiusTopRight: =12
            RadiusBottomLeft: =12
            RadiusBottomRight: =12
```
