# Spacing & Typography Reference

## Spacing Scale

All values in pixels. Use these for padding, margin, gap, width, height.

```
4   8   12   16   20   24   32   40   48   64   80   96   128   160   192   256
```

**Tailwind equivalents:**
| px | Tailwind | px | Tailwind |
|----|----------|----|----------|
| 4  | 1        | 40 | 10       |
| 8  | 2        | 48 | 12       |
| 12 | 3        | 64 | 16       |
| 16 | 4        | 80 | 20       |
| 20 | 5        | 96 | 24       |
| 24 | 6        | 128| 32       |
| 32 | 8        | 160| 40       |

## Common Spacing Patterns

| Element | Padding | Gap between |
|---------|---------|-------------|
| Button (sm) | 6px 12px | — |
| Button (md) | 10px 20px | — |
| Button (lg) | 14px 28px | — |
| Input field | 10px 14px | — |
| Card | 24-32px | 16-24px grid gap |
| Section | 48-96px top/bottom | — |
| Sidebar | 16-24px | — |
| Nav items | — | 8-16px |
| Form fields | — | 16-24px |
| Form groups | — | 32-48px |
| List items | 8-12px vertical | — |
| Page container | 16-24px horizontal (mobile), max-width center (desktop) |

## Font Size Scale

```
12   14   16   18   20   24   30   36   48   60   72
```

| Size | CSS rem | Use for |
|------|---------|---------|
| 12px | 0.75rem | Fine print, labels, badges, captions |
| 14px | 0.875rem | Secondary text, table cells, small UI |
| 16px | 1rem | Body text (default), input values |
| 18px | 1.125rem | Lead paragraphs, slightly emphasized body |
| 20px | 1.25rem | Card titles, section subheads |
| 24px | 1.5rem | Section headings (h3) |
| 30px | 1.875rem | Page subheadings (h2) |
| 36px | 2.25rem | Page titles (h1) |
| 48px | 3rem | Hero headings |
| 60px | 3.75rem | Marketing/landing hero |
| 72px | 4.5rem | Large display text |

## Line Height

| Context | Line-height | Why |
|---------|-------------|-----|
| Headings (24px+) | 1.2 - 1.25 | Large text needs tight leading |
| Body text (14-18px) | 1.5 - 1.75 | Comfortable reading |
| Tight UI (labels, buttons) | 1 - 1.25 | Space efficiency |
| Wide text blocks (>60ch) | 1.75 | Eyes need help finding next line |
| Narrow columns (<40ch) | 1.4 - 1.5 | Less travel distance |

## Letter Spacing

| Context | Letter-spacing |
|---------|---------------|
| Large headings (36px+) | -0.025em (tighter) |
| Body text | 0 (normal) |
| All-caps labels/badges | +0.05em to +0.1em (wider) |
| Small text (12px) | +0.01em (slightly wider for legibility) |

## Font Recommendations

Recommended font families from Refactoring UI:

### Sans-Serif (UI, Headings, General)
- **Inter** — Excellent for UI, optimized for screens, free
- **Helvetica Neue** — Classic, clean, professional
- **Proxima Nova** — Geometric, modern, versatile
- **Aktiv Grotesk** — Professional alternative to Helvetica
- **Acumin Pro** — Adobe's Helvetica competitor, clean
- **Roboto** — Google's system font, very readable
- **Source Sans Pro** — Adobe, great for body text, free
- **Open Sans** — Highly readable, neutral, free
- **IBM Plex Sans** — Modern, technical feel, free
- **Nunito Sans** — Rounded, friendly, free

### Serif (Long-form content, editorial)
- **Georgia** — System serif, great for body
- **Lora** — Elegant, readable, free
- **Merriweather** — Designed for screens, free
- **Playfair Display** — High-contrast, editorial feel, free
- **Source Serif Pro** — Pairs with Source Sans Pro, free

### Monospace (Code, data)
- **JetBrains Mono** — Designed for code, excellent ligatures, free
- **Fira Code** — Ligatures for programming, free
- **Source Code Pro** — Adobe, clean, free
- **IBM Plex Mono** — Pairs with IBM Plex Sans, free

### Pairing Strategy
One font family is usually enough for most UIs. Create contrast through
size and weight variations, not multiple typefaces. If pairing:
- Use sans-serif for navigation, UI controls, short text
- Use serif for long-form body content
- Never pair two sans-serifs or two serifs

## Max-Width Reference

| Content type | Max-width | Tailwind |
|-------------|-----------|----------|
| Narrow text/forms | 540-640px | max-w-xl / max-w-2xl |
| Blog/article | 680-768px | max-w-3xl |
| Content + sidebar | 1024px | max-w-5xl |
| Dashboard/app | 1280px | max-w-7xl |
| Full-width with padding | 1440px | max-w-screen-2xl |

## Border Radius Reference

| Element | Radius | Feel |
|---------|--------|------|
| Buttons | 4-6px | Professional |
| Buttons | 8px | Friendly |
| Buttons | 9999px (full) | Playful |
| Cards | 8-12px | Standard |
| Inputs | 4-6px | Standard |
| Avatars | 50% (circle) | Standard |
| Modals | 12-16px | Modern |
| Badges/pills | 9999px (full) | Standard |
