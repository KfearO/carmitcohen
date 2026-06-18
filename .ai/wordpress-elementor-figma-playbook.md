
# WordPress + Elementor + Figma Playbook

# Scope

This document is not a record of a specific project.
It is a collection of principles, patterns and lessons learned from real projects.
The goal is to improve consistency, maintainability and client editability while staying as close as possible to the original Figma design.
This playbook should evolve over time.

## Goal

Build websites from Figma using WordPress + Elementor while maximizing maintainability for non-technical clients and minimizing custom CSS.

---

# Philosophy

## Documentation Philosophy

Do not document values.
Document patterns.
Values are temporary.
Patterns are reusable.

Bad:
```
Button width = 230px
Top padding = 60px
```

Good:

```
Use Elementor controls.
Prefer structure over CSS.
Think in containers.
Prefer SVG over CSS shapes.
```

The purpose of this playbook is not to preserve implementation details.

The purpose is to preserve principles and patterns that remain useful across projects.

---

Prefer:

1. Elementor settings.
2. Better container structure.
3. Additional containers when needed.
4. SVG assets exported from Figma.
5. CSS as a last resort.

Never start with CSS.
The goal is not merely to reproduce the Figma design.
The goal is to create a maintainable website that clients can edit themselves.

---

# Success Criteria

Aim for the closest possible result to the Figma design.
Do not introduce visible differences unless there is a clear technical reason.
Success means:

* Visually as close as possible to the design.
* Editable by the client.
* Zero CSS when possible; otherwise the minimum necessary CSS.
* Maintainable structure.

Small typography differences between Figma and browsers can happen, but they should be treated as something to verify, not as a default excuse.

---

# General Principles

## Think in containers, not coordinates

Bad mindset:

- Absolute positioning everywhere.
- Pixel-perfect coordinates.
- Solving everything with CSS.

Good mindset:

- Parent containers.
- Nested containers.
- Flexbox.
- Space Between.
- Logical structure.

---

# Elementor Versions Evolve

Elementor UI changes between versions.

Do not rely on the exact location of controls.

Prefer understanding concepts:

```
Page Layout
Container hierarchy
Flexbox
Spacing
Theme interaction
```

rather than memorizing specific buttons or icons.

Examples:

* Settings panels may move.
* Some controls may disappear.
* New UI versions can replace old workflows.

Always verify the current Elementor interface before assuming a feature no longer exists.

---

# Simplify Content Before Adding Complexity

Sometimes changing content is better than adding structure.

Bad:

```
Extra containers
Min heights
Special CSS
```

Good:

```
IKEBANA DEMO
```

instead of:

```
IKEBANA DEMONSTRATION
```

Prefer simpler content over more technical complexity.

---

# SVG Strategy

## Prefer SVG over CSS shapes

Bad:

```text
.green-circle {
    ...
}

.blue-shape {
    ...
}
```

Good:

```
Figma
↓
Export SVG
↓
Upload to Elementor
↓
Position visually
```

Advantages:

* Client editable.
* No CSS.
* Easier maintenance.
* Closer to Figma.

---

# SVG Export Notes

Some SVG filters do not work correctly inside WordPress / Elementor.

Problematic elements:

```
<filter>
<defs>
<clipPath>
```

If needed:

* Remove filters.
* Simplify SVG files.
* Export again.

---

# Typography

Prefer Elementor typography controls.

Avoid CSS whenever possible.

Instead of:

```text
line-height: 1.15;
margin: 0;
```

Use:

```
Typography
    Line Height
```

inside Elementor.

If:

* Font family matches.
* Font weight matches.
* Size matches.
* Line height matches.
* Chrome reports the expected rendered font.

Then remaining differences are usually rendering-engine differences.
Do not jump to CSS hacks.
First, verify that the browser is rendering the correct font with the correct settings.
If a difference remains after that, document it clearly before deciding to accept it.

---

# Widths

Avoid CSS widths.

Bad:

```css
.elementor-button {
    width: 230px;
}
```

Good:

```
Elementor controls
↓
Position
↓
Stretch
```

This is a pattern example, not a value to memorize.
Document the decision path, not the specific numeric value.

Do not assume that Custom Width affects the element you actually want.
Widget width and content width are not always the same.
Explore Elementor controls before introducing CSS.

---

# Container Structure Matters

Many CSS problems are actually structure problems.

---

## Bad Structure

```
service-card
├── background-area
├── content-area
└── button
```

This makes "Space Between" unusable.

---

## Good Structure

```
service-card
├── upper-area
│   ├── background-area
│   └── content-area
│
└── button
```

Settings:

```
service-card
    Direction = Column
    Justify Content = Space Between

upper-area
    Direction = Column
    Justify Content = Start
```

This allows the button to remain at the bottom while keeping the text grouped correctly.

---

# Remove Containers Aggressively

Containers used only for spacing should be questioned.

Example:

```
title-area
```

it was removed completely.

Spacing was restored using Elementor controls.

Fewer containers usually mean:

* Better alignment.
* Less CSS.
* Easier maintenance.

---

# Padding vs. Margin

Prefer:

```
Padding
```

for internal spacing.

Use:

```
Margin
```

only for spacing between elements.

---

# Absolute Elements

Decorative SVGs can use:

```
Position = Absolute
```

while the surrounding layout remains Flexbox-based.

---

# Space Between

Space Between is powerful.

If it breaks the layout:
Do not immediately use CSS.

First ask:
> Do I need another container?

Very often the answer is yes.

---

# CSS Policy

Before writing CSS ask:

1. Can Elementor do this?
2. Can a better structure solve it?
3. Can another container solve it?
4. Can SVG solve it?

Only then write CSS.

---

# Backup Strategy

Before experimenting:

1. Export Elementor templates.
2. Save custom CSS.
3. Commit to Git.

For simple projects with no dynamic data, database backups are optional.

---

# Elementor Warning

Do not use:

```
Back to WordPress Editor
```

on Elementor-built pages.

If problems occur:

```
Elementor
    → History
    → Revisions
```

can usually restore the page.

---

# One More Principle

When solving a visual problem, preserve the design first.
Only change the design when the technical cost is clearly unjustified and the tradeoff is deliberate.

Prefer:

```
Simpler content
↓
Better structure
↓
Additional containers
↓
SVG assets
↓
CSS
```

rather than the opposite.

---

# Key Lesson

When Elementor appears to require CSS, the real problem is often the container hierarchy.
Good structure beats custom CSS.
