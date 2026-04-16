---
name: canvas-responsive-layout
description: Design and generate responsive layout YAML code for Microsoft Power Apps Canvas Apps. Use this skill whenever a user wants to build a responsive app layout, create a canvas app screen, design a UI with containers or shapes, generate YAML for Power Apps Studio, or asks about responsive design patterns in Power Apps. Even if the user just says "make me a sidebar layout" or "build a Power Apps screen with a header and content area", invoke this skill — it knows the exact YAML syntax needed and will produce paste-ready code.
---

# Canvas Responsive Layout Designer

You design responsive canvas app layouts and output YAML code that can be pasted directly into Power Apps Studio.

## Reference files

Read these when you need detailed information:

| File | When to read |
|------|-------------|
| `references/yaml-patterns.md` | **Read first** — exact YAML syntax, control types, property values, and complete layout patterns |
| `references/build-responsive-apps.md` | Auto-layout container concepts, when to use horizontal vs vertical, known issues |
| `references/create-responsive-layout.md` | Parent.Width/Height formulas, breakpoints, orientation handling |
| `references/control-horizontal-container.md` | Horizontal container property reference |
| `references/control-vertical-container.md` | Vertical container property reference |
| `references/control-container.md` | Manual layout container reference |
| `references/control-shapes-icons.md` | Shape and Icon control properties |

**Always read `references/yaml-patterns.md` before generating any YAML.** It has the exact control type names and variant strings you need.

---

## Your workflow

### 1. Understand the layout

Ask clarifying questions only if the request is genuinely ambiguous. Most layout requests can be interpreted and executed directly. Typical things to infer:

- **Layout type**: header+body, sidebar+content, split-screen, card grid, full-screen, dashboard, form
- **Number of sections** and their relative sizes (e.g., sidebar is ~1/4 width, main is ~3/4)
- **Scrolling behavior**: which sections should scroll
- **Color scheme**: if not specified, use a clean neutral palette (white body, blue header/sidebar accents)
- **Nesting**: cards inside a scrollable body, a toolbar inside a header, etc.

### 2. Design the container hierarchy

Think through the tree of containers before writing YAML. A good mental model:

```
Screen
└── Root (vertical, fills screen)
    ├── Header (horizontal, fixed height)
    │   ├── Logo/Title area
    │   └── Action icons area
    └── Body (horizontal or vertical, FillPortions=1)
        ├── Sidebar (vertical, fixed width)
        └── Content (vertical, FillPortions=1, scrollable)
```

The key insight: **auto-layout containers eliminate X/Y positioning**. You only need to think about:
- Direction (horizontal stacks children left-to-right; vertical stacks top-to-bottom)
- How children share space (FillPortions for flexible, fixed Width/Height for rigid)
- Alignment on the cross axis (LayoutAlignItems)
- Spacing (LayoutGap, Padding)

### 3. Generate the YAML

Produce complete, paste-ready YAML. Structure it as a list of controls (no `Screens:` wrapper) — this is what gets pasted into Canvas Apps Studio.

Always include:
- Descriptive control names (not `GroupContainer1` — use `AppShell`, `NavSidebar`, `HeaderBar`, etc.)
- The correct `Variant` for auto-layout containers
- `FillPortions: =0` explicitly on fixed-size children (with a corresponding Width or Height)
- `FillPortions: =1` on flexible children
- `AlignInContainer: =AlignInContainer.Stretch` on children that should fill the cross axis

### 4. Explain the structure

After the YAML, briefly explain:
- The container hierarchy
- How it adapts to screen size changes
- Any key design decisions (e.g., why a sidebar is fixed-width instead of proportional)

---

## Core responsive design rules

1. **Root container fills the screen**: `X: =0`, `Y: =0`, `Width: =Parent.Width`, `Height: =Parent.Height`

2. **Children of auto-layout containers don't need X/Y** — the container positions them automatically

3. **FillPortions drives proportional sizing** along the primary axis:
   - `FillPortions: =1` on all three siblings → equal thirds
   - `FillPortions: =1` + `FillPortions: =3` → 25% / 75% split

4. **Fixed-size items** use `FillPortions: =0` plus an explicit `Width` (in horizontal containers) or `Height` (in vertical containers)

5. **Scrollable regions** need `LayoutOverflowY: =LayoutOverflow.Scroll` (in a vertical container) or `LayoutOverflowX: =LayoutOverflow.Scroll` (in a horizontal container), and their size must be bounded by a parent giving them a finite height

6. **Shapes inside auto-layout containers**: don't set X/Y; set Width and Height explicitly, or use FillPortions for proportional sizing. Use `AlignInContainer` for positioning on the off-axis.

7. **Screen-size responsiveness** comes from `Parent.Width`/`Parent.Height` at the root, then flowing down through FillPortions — no hardcoded pixel sizes at the outer levels

---

## Container quick reference

| Need | Use |
|------|-----|
| Stack children top-to-bottom | Vertical container |
| Stack children left-to-right | Horizontal container |
| Overlap children freely | Manual layout container |
| Equal-width columns | Horizontal + `FillPortions: =1` on each child |
| Fixed sidebar + flexible content | Horizontal + sidebar `FillPortions: =0, Width: =240` + content `FillPortions: =1` |
| Fixed header + scrollable body | Vertical + header `FillPortions: =0, Height: =64` + body `FillPortions: =1, LayoutOverflowY: =LayoutOverflow.Scroll` |
| Centered content | Set `LayoutAlignItems: =LayoutAlignItems.Center` and `LayoutJustifyContent: =LayoutJustifyContent.Center` |
| Evenly spaced items | `LayoutJustifyContent: =LayoutJustifyContent.SpaceBetween` |
| Items with gap | `LayoutGap: =16` (or whatever pixel value) |

---

## Shape quick reference

Shapes are visual elements (rectangles, circles, etc.) that can be placed inside containers:

```yaml
- MyRect:
    Control: Rectangle@2.3.0
    Properties:
      Fill: =RGBA(56, 96, 178, 1)
      Width: =Parent.Width    # or a fixed pixel value like =120
      Height: =4              # e.g., a divider line
```

Inside an auto-layout container, shapes participate in the layout flow:
- Their `Width`/`Height` can be fixed pixels or `Parent.Width`/`Parent.Height`
- Use `AlignInContainer` to override the parent's default alignment
- Use `FillPortions: =1` to make a shape stretch to fill available space

---

## ⚠️ Common mistakes to avoid

These are errors that occur without the skill's guidance — **never produce these**:

| ❌ Wrong | ✅ Correct | Why |
|---------|-----------|-----|
| `Control: GroupContainer` | `Control: GroupContainer@1.5.0` | Must include correct version |
| `Control: GroupContainer@0.0.44` | `Control: GroupContainer@1.5.0` | `@0.0.44` is outdated |
| `Control: GroupContainer@2.0.0` | `Control: GroupContainer@1.5.0` | `@2.0.0` does not exist |
| `Control: Rectangle@0.0.44` | `Control: Rectangle@2.3.0` | Current version is `2.3.0` |
| `Control: Circle@0.0.44` | `Control: Circle@2.3.0` | Current version is `2.3.0` |
| `Variant: verticalAutoLayoutContainer` | `Variant: AutoLayout` + `LayoutDirection: =LayoutDirection.Vertical` | Correct variant is `AutoLayout`; direction is a separate property |
| `Variant: horizontalAutoLayoutContainer` | `Variant: AutoLayout` + `LayoutDirection: =LayoutDirection.Horizontal` | Correct variant is `AutoLayout`; direction is a separate property |
| `Variant: manualLayoutContainer` | `Variant: ManualLayout` | Correct variant name is `ManualLayout` |
| `AlignItems: =AlignItems.Stretch` | `LayoutAlignItems: =LayoutAlignItems.Stretch` | Correct property name is `LayoutAlignItems` |
| `Gap: =16` | `LayoutGap: =16` | Correct property name is `LayoutGap` |
| `Width: =App.Width * 0.6` | `FillPortions: =3` (sibling at `FillPortions: =2`) | Use FillPortions, not percentage math |
| `Width: =App.Width` | `Width: =Parent.Width` | Use `Parent`, not `App` for sizing |
| `Screens:` wrapper around output | Bare list of controls (no wrapper) | Output must be pasteable directly — no `Screens:` wrapper |
| X/Y set on auto-layout children | Omit X/Y inside auto-layout containers | Auto-layout containers manage child positions |
| `RadiusTopLeft` on `Rectangle@2.3.0` | Use `GroupContainer@1.5.0` with `Variant: ManualLayout` for rounded shapes | `Rectangle` does not support radius properties |
| `Width: =Parent.Width * 0.7` on a ManualLayout child inside AutoLayout (progress track ≤4px) | Use sibling `FillPortions` ratio or make track ≥8px tall | Tiny ManualLayout fills may not render visibly at low heights |

---

## Validated complex patterns (live-tested in Power Apps Studio)

The following patterns are confirmed working in a full analytics dashboard with 6 nesting levels:

- **Bar chart with bottom-aligned variable-height columns**: Set `LayoutAlignItems: =LayoutAlignItems.End` on the columns container + `AlignInContainer: =AlignInContainer.End` + `Variant: ManualLayout` + explicit `Height` on each bar. Produces bottom-anchored bars of varying heights.

- **Pill/badge (status chip)**: `GroupContainer@1.5.0` `Variant: ManualLayout` with all four `Radius*: =12` produces a pill. Confirmed working as a child of AutoLayout rows.

- **Data table rows**: Horizontal AutoLayout with `FillPortions` columns (some =2 for name, some =1 for data, =0 for fixed badge width). Badge cell uses ManualLayout with radius for rounded pill appearance.

- **KPI cards in a grid row**: 4 equal `FillPortions: =1` cards in a horizontal AutoLayout, each a vertical AutoLayout card with value rectangle + label rectangle.

See `references/yaml-patterns.md` (bottom section) for full YAML for each of these patterns.

---

## Validation approach

When the user says they've pasted the YAML and will share screenshots:

1. Ask them to share the screenshot
2. Check: Does the layout match the described intent? Are sections the right proportions?
3. If something looks wrong, diagnose the most likely cause:
   - Missing `Stretch` alignment = children not expanding to fill the container
   - Wrong FillPortions = proportions off
   - Overflow not set = content clipping instead of scrolling
   - Fixed Width/Height on flexible child = not adapting to screen size
4. Provide corrected YAML for just the affected portion

---

## Example: Sidebar app with colored shapes

This example shows a classic sidebar layout using only containers and shapes.

**User prompt:** "Create a sidebar layout with a blue nav panel on the left (25% width) and a white content area. Add a header bar at the top."

**Output:**

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
      - HeaderBar:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
            LayoutDirection: =LayoutDirection.Horizontal
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =0
            Height: =64
            LayoutAlignItems: =LayoutAlignItems.Center
            LayoutJustifyContent: =LayoutJustifyContent.Start
            LayoutGap: =16
            PaddingLeft: =16
            Fill: =RGBA(56, 96, 178, 1)
          Children:
            - HeaderAccent:
                Control: Rectangle@2.3.0
                Properties:
                  Fill: =RGBA(255,255,255,0.2)
                  Width: =4
                  Height: =40
      - MainArea:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
            LayoutDirection: =LayoutDirection.Horizontal
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =1
            LayoutAlignItems: =LayoutAlignItems.Stretch
            Fill: =RGBA(0,0,0,0)
          Children:
            - NavSidebar:
                Control: GroupContainer@1.5.0
                Variant: AutoLayout
            LayoutDirection: =LayoutDirection.Vertical
                Properties:
                  AlignInContainer: =AlignInContainer.Stretch
                  FillPortions: =1
                  LayoutAlignItems: =LayoutAlignItems.Stretch
                  LayoutJustifyContent: =LayoutJustifyContent.Start
                  LayoutGap: =8
                  PaddingTop: =16
                  PaddingLeft: =8
                  PaddingRight: =8
                  Fill: =RGBA(33, 37, 41, 1)
                Children:
                  - NavItem1:
                      Control: Rectangle@2.3.0
                      Properties:
                        AlignInContainer: =AlignInContainer.Stretch
                        FillPortions: =0
                        Height: =44
                        Fill: =RGBA(255,255,255,0.1)
                        RadiusTopLeft: =6
                        RadiusTopRight: =6
                        RadiusBottomLeft: =6
                        RadiusBottomRight: =6
                  - NavItem2:
                      Control: Rectangle@2.3.0
                      Properties:
                        AlignInContainer: =AlignInContainer.Stretch
                        FillPortions: =0
                        Height: =44
                        Fill: =RGBA(255,255,255,0.05)
                        RadiusTopLeft: =6
                        RadiusTopRight: =6
                        RadiusBottomLeft: =6
                        RadiusBottomRight: =6
                  - NavItem3:
                      Control: Rectangle@2.3.0
                      Properties:
                        AlignInContainer: =AlignInContainer.Stretch
                        FillPortions: =0
                        Height: =44
                        Fill: =RGBA(255,255,255,0.05)
                        RadiusTopLeft: =6
                        RadiusTopRight: =6
                        RadiusBottomLeft: =6
                        RadiusBottomRight: =6
            - ContentArea:
                Control: GroupContainer@1.5.0
                Variant: AutoLayout
            LayoutDirection: =LayoutDirection.Vertical
                Properties:
                  AlignInContainer: =AlignInContainer.Stretch
                  FillPortions: =3
                  LayoutAlignItems: =LayoutAlignItems.Stretch
                  LayoutJustifyContent: =LayoutJustifyContent.Start
                  LayoutGap: =16
                  PaddingTop: =24
                  PaddingLeft: =24
                  PaddingRight: =24
                  LayoutOverflowY: =LayoutOverflow.Scroll
                  Fill: =RGBA(248, 249, 250, 1)
                Children:
                  - ContentCard:
                      Control: GroupContainer@1.5.0
                      Variant: AutoLayout
            LayoutDirection: =LayoutDirection.Vertical
                      Properties:
                        AlignInContainer: =AlignInContainer.Stretch
                        FillPortions: =0
                        Height: =120
                        LayoutAlignItems: =LayoutAlignItems.Center
                        LayoutJustifyContent: =LayoutJustifyContent.Center
                        Fill: =RGBA(255,255,255,1)
                        RadiusTopLeft: =8
                        RadiusTopRight: =8
                        RadiusBottomLeft: =8
                        RadiusBottomRight: =8
```

This creates:
- **AppShell**: vertical stack filling the whole screen
- **HeaderBar**: fixed 64px tall, dark blue, horizontal
- **MainArea**: horizontal, takes remaining height (FillPortions=1)
  - **NavSidebar**: 25% width (FillPortions=1 of 4 total), dark background, with rectangle nav items
  - **ContentArea**: 75% width (FillPortions=3 of 4 total), light gray background, scrollable
