
# WordPress + Elementor + Figma Playbook

## Goal

Build websites from Figma using WordPress + Elementor while maximizing maintainability for non-technical clients and minimizing custom CSS.

---

# Philosophy

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

# POC Result

Final result:

```

0 lines of CSS

```

This was achieved without compromising the design.

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

---

# Widths

Avoid CSS widths.

Bad:

```css
.elementor-button {
    width: 230px;
}
```

Prefer Elementor controls.

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
│
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

# Padding vs Margin

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

# Key Lesson

When Elementor appears to require CSS, the real problem is often the container hierarchy.

Good structure beats custom CSS.


