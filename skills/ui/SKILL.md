---
name: ui
description: >
  UI design system skill based on Refactoring UI principles. Produces
  professional-looking frontend code with proper visual hierarchy, spacing
  scales, curated color palettes, typography, shadows, and layout.
  TRIGGER when: writing HTML/CSS/Tailwind, building UI components, styling
  pages or layouts, choosing colors or spacing, creating forms/cards/navbars/
  dashboards, or any frontend code that will be seen by users. Also trigger
  when the user says things like "make it look good", "improve the styling",
  "it looks amateur", "fix the design", "pick colors", or asks about spacing,
  font sizes, or visual hierarchy. Use this skill even for small styling tasks
  like button styling or card layouts — the principles apply at every scale.
  DO NOT TRIGGER when: writing backend code, APIs, database queries, or
  non-visual logic with no UI component.
---

# UI Design System — Refactoring UI Principles

This skill encodes the design system from Refactoring UI (Adam Wathan &
Steve Schoger). Apply these principles whenever generating frontend code
to produce UIs that look designed, not default.

**Related skills:** This skill is framework-agnostic and pairs with any
frontend framework skill — `solidjs`, `shadcn` (component patterns for any
web framework), and `react-native` (for mobile). The scales and principles
here are the visual design baseline; framework skills handle framework-specific
idioms.

The single biggest thing that separates "designed" from "amateur" is
intentional constraint. Amateur UIs use arbitrary values — random padding,
any hex color, whatever font-size feels right. Professional UIs use
predefined scales and systems. That's what this skill provides.

## Quick Reference: The Scales

Before diving into principles, here are the core scales to use in every project.

### Spacing Scale (px)

Use these values for all padding, margin, gap, and sizing:

```
4  8  12  16  20  24  32  40  48  64  80  96  128  160  192  256
```

In Tailwind: `1  2  3  4  5  6  8  10  12  16  20  24  32  40  48  64`

Pick values from this scale — never use arbitrary numbers like 13px or 37px.
The constraint is the point. If 16px feels too small and 20px too big, pick
one and move on. Consistency beats pixel-perfection.

### Font Size Scale (px)

```
12  14  16  18  20  24  30  36  48  60  72
```

Body text: 16-18px. UI labels: 12-14px. Headings: 24-48px.
Two sizes that are close (14 and 15) waste a slot — make jumps meaningful.

### Shadow Scale

Five levels, from subtle to dramatic:

```css
--shadow-xs:  0 1px 2px rgba(0,0,0,0.05);
--shadow-sm:  0 1px 3px rgba(0,0,0,0.1), 0 1px 2px rgba(0,0,0,0.06);
--shadow-md:  0 4px 6px rgba(0,0,0,0.07), 0 2px 4px rgba(0,0,0,0.06);
--shadow-lg:  0 10px 15px rgba(0,0,0,0.1), 0 4px 6px rgba(0,0,0,0.05);
--shadow-xl:  0 20px 25px rgba(0,0,0,0.1), 0 10px 10px rgba(0,0,0,0.04);
```

xs: subtle depth for cards at rest. sm: buttons, inputs. md: dropdowns,
popovers. lg: modals, dialogs. xl: large floating panels.

## 1. Visual Hierarchy

The most impactful design principle. Not every element deserves attention —
deliberately assign importance through size, weight, and color.

**Three tiers of emphasis:**

| Tier | Font weight | Color | Use for |
|------|------------|-------|---------|
| Primary | 600-700 (semibold/bold) | grey-900 / near-black | Headings, key data, CTAs |
| Secondary | 400-500 (normal/medium) | grey-500-600 | Supporting text, descriptions |
| Tertiary | 400 (normal) | grey-400 | Timestamps, metadata, labels |

Font size alone is a crude tool for hierarchy. Weight and color are often
more effective — you can make secondary text the same size as primary text
but in grey-500 medium weight, and the hierarchy is clear without making
anything tiny or enormous.

**Labels are often unnecessary.** If a card shows "John Smith" in bold and
"john@example.com" below it in grey, you don't need "Name:" and "Email:"
labels — the format tells the user what they're looking at. Remove labels
when the data is self-evident; add them only when values would be ambiguous
without context.

**De-emphasize to emphasize.** When everything is bold and dark, nothing
stands out. The most effective way to make important elements pop is to
tone down everything around them. A bold heading next to muted grey text
has more impact than making the heading even larger.

## 2. Spacing and Layout

Start with more whitespace than feels comfortable, then reduce if needed.
The most common amateur mistake is cramming elements too close together.

**Component internal spacing:** Use generous padding. A card that feels
"boxy" probably needs 24-32px padding, not 12-16px. Buttons need horizontal
padding 2-3x their vertical padding (e.g., 12px 24px, not 12px 16px).

**Between-element spacing:** Space between related items (e.g., list items)
should be smaller than space between groups. Use the spacing scale — 8-12px
between related items, 24-32px between groups, 48-64px between sections.

**Max-width matters.** Full-width text on a large screen is unreadable.
Constrain content width:
- Narrow text (articles, forms): 540-640px (32-40rem)
- Medium content: 768px (48rem)
- Wide content: 1024-1280px (64-80rem)

Body text line length should be 45-75 characters (~20-35em). If it's wider,
readers lose their place jumping to the next line.

**Don't make everything fluid.** Some elements (avatars, icons, badges)
should be fixed-size. Sidebars can be fixed-width while the main content
flexes. Not every dimension needs a percentage.

## 3. Color

Read `references/color-palettes.md` for the 24 curated palettes and selection guide.
Read `references/color-swatches.md` for the full hex value catalog.

**What a complete color palette needs:**
- **Greys** (8-10 shades): the backbone — used for text, backgrounds, borders, dividers
- **Primary brand color** (8-10 shades): buttons, links, active states, accents
- **Semantic colors**: danger/red, warning/yellow, success/green, info/blue — each 5-10 shades

When picking colors, choose from the curated palettes in the references. Each
palette has a primary, neutral (grey family), and supporting colors that are
designed to work together.

**How to apply shades:**
- Dark text on light bg: grey-700 to grey-900 (never pure #000)
- Secondary text: grey-400 to grey-600
- Placeholder text: grey-300 to grey-400
- Subtle borders: grey-200 to grey-300
- Subtle backgrounds: grey-050 to grey-100 (alternating rows, hover states)
- Primary button: primary-500 to primary-600 (bg), white (text)
- Primary button hover: primary-600 to primary-700
- Danger button: red-500 to red-600

**On colored backgrounds:** Don't use grey text — it looks washed out.
Instead, use a lighter/darker shade of the background color itself. For
example, on a blue-600 background, use blue-100 for secondary text, not grey-400.

**Accessibility:** Ensure 4.5:1 contrast ratio for normal text, 3:1 for
large text (18px+ bold or 24px+ normal). The shade pairings above generally
meet this. When in doubt, pair -700/-800/-900 text with -050/-100 backgrounds.

## 4. Typography

**Line-height:** Proportional to font size and line length.
- Body text (14-18px): line-height 1.5-1.75
- Headings (24px+): line-height 1.2-1.25
- Tight UI text (labels, buttons): line-height 1-1.25

Wider text blocks need more line-height. Narrow columns can use less.

**Letter-spacing:**
- Large headings (36px+): slightly tighter (-0.025em)
- Body text: normal (0)
- All-caps text (labels, badges): wider (+0.05em to +0.1em)

**Alignment:** Left-align by default. Center-align only for short text
(headings, CTAs, hero sections — never more than 2-3 lines). Right-align
only for numbers in tables.

**Font pairing:** One font family is usually enough. Create contrast through
size and weight, not multiple typefaces. If you do pair, use a sans-serif
for UI and a serif for long-form content — don't use two sans-serifs.

Read `references/spacing-typography.md` for font family recommendations.

## 5. Shadows and Depth

Shadows communicate elevation and interactivity — use them intentionally.

**Elevation model:** Higher elements cast larger, more diffused shadows.
A card flat on the page gets shadow-xs. A dropdown floating above it gets
shadow-md. A modal over everything gets shadow-lg.

**Interactive feedback:** Buttons can shift shadow on interaction:
- Default: shadow-sm
- Hover: shadow-md (lifts up)
- Active/pressed: shadow-xs (presses down)

**Two-shadow technique:** Combine a tight, dark shadow with a larger, soft
one for natural-looking depth:
```css
box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.08);
```

**Colored shadows:** On colored elements, tint the shadow with the element's
color. A blue button looks more natural with `rgba(37,99,235,0.3)` shadow
than `rgba(0,0,0,0.3)`.

## 6. Borders and Separation

Borders are the most overused UI element. Before adding a border, try:

1. **More spacing** — padding or gap often creates enough separation
2. **Different backgrounds** — alternate bg colors (white vs grey-050)
3. **Box shadow** — shadow-xs creates a subtle boundary without a hard line

When you do need a border, keep it subtle: 1px solid grey-200. Heavy or
dark borders create visual noise.

**Accent borders** are an exception — a 4px left or top border in a brand
color adds personality. Use on cards, alerts, sidebar sections, or active
nav items. This is one of the simplest finishing touches.

## 7. Buttons

Buttons need a clear hierarchy — not every action is equally important.

| Style | Use for | Example |
|-------|---------|---------|
| **Solid/filled** | Primary action (1 per section) | "Save changes", "Submit" |
| **Outline** | Secondary actions | "Cancel", "Export" |
| **Ghost/text** | Tertiary actions | "Learn more", "Skip" |
| **Destructive** | Dangerous actions (red solid) | "Delete account" |

Sizing: vertical padding 8-12px, horizontal padding 16-32px (wider looks
better). Border-radius: 4-8px for professional, fully round for playful.

## 8. Forms

- Labels above inputs (not beside — that breaks on mobile)
- Input height: 40-48px (comfortable click target)
- Input padding: 10-14px vertical, 12-16px horizontal
- Border: 1px solid grey-300, focus: 2px solid primary-500 (or ring)
- Placeholder text: grey-400, never as a replacement for labels
- Error state: red-500 border + red-600 text message below
- Group related fields, separate groups with 24-32px space
- Keep forms narrow (max 480-540px) — wide inputs look wrong

## 9. Cards

- Padding: 24-32px (generous)
- Border-radius: 8-12px
- Background: white on grey-050 page, or grey-050 on white page
- Prefer shadow-xs or shadow-sm over borders for card boundaries
- Image at top: full-bleed (no padding), content below with padding
- Card grids: 16-24px gap between cards

## Applying These Principles

When generating frontend code:

1. **Pick a palette first.** Read `references/color-palettes.md`, choose one
   that fits the project, and use its colors consistently throughout.
2. **Use the spacing scale.** Every margin, padding, and gap should come from
   the scale. If using Tailwind, stick to the default spacing utilities.
3. **Establish hierarchy.** For every component, decide what's primary,
   secondary, and tertiary. Style accordingly using weight and color.
4. **Start spacious.** Default to generous whitespace. It's easier to tighten
   later than to pry apart cramped elements.
5. **Limit variation.** Use 2-3 font sizes, 2-3 grey shades, and 1-2 accent
   colors per component. Constraint looks intentional; variety looks chaotic.

## CSS Custom Properties Template

When setting up a new project, define these variables:

```css
:root {
  /* Pick from references/color-palettes.md */
  --color-primary-50: /* lightest */;
  --color-primary-500: /* base */;
  --color-primary-700: /* dark */;
  --color-grey-50: /* lightest bg */;
  --color-grey-200: /* borders */;
  --color-grey-400: /* secondary text */;
  --color-grey-700: /* primary text */;
  --color-grey-900: /* headings */;

  /* Spacing uses the scale directly: 4, 8, 12, 16, 20, 24, 32... */

  /* Typography */
  --text-xs: 0.75rem;   /* 12px */
  --text-sm: 0.875rem;  /* 14px */
  --text-base: 1rem;    /* 16px */
  --text-lg: 1.125rem;  /* 18px */
  --text-xl: 1.25rem;   /* 20px */
  --text-2xl: 1.5rem;   /* 24px */
  --text-3xl: 1.875rem; /* 30px */
  --text-4xl: 2.25rem;  /* 36px */
}
```
