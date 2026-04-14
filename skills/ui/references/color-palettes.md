# Color Palettes — Refactoring UI

24 curated palettes designed for real UIs. Each palette has a **primary** color
(brand identity), **neutrals** (greys for text/backgrounds), and **supporting**
colors (accents, semantic states). All colors come from the swatches in
`color-swatches.md` — refer there for exact hex values.

## How to Choose a Palette

1. Start with the **primary hue** that fits the project's personality
2. The neutral grey family sets the overall feel (cool = techy, warm = friendly)
3. Supporting colors handle accents, errors, warnings, success states

## Palette Index

| # | Primary | Neutral | Supporting | Character |
|---|---------|---------|------------|-----------|
| 1 | Cyan | Blue-Grey | Indigo, Pink, Red, Yellow | Clean, professional |
| 2 | Blue | Blue-Grey | Cyan, Red, Yellow-Vivid | Corporate, trustworthy |
| 3 | Purple | Blue-Grey | Light-Blue-Vivid, Red-Vivid, Teal, Yellow-Vivid | Creative, modern |
| 4 | Teal | Blue-Grey | Blue, Purple, Red, Yellow | Fresh, balanced |
| 5 | Blue-Grey | Blue-Grey | Light-Blue-Vivid, Cyan, Pink-Vivid, Red-Vivid, Yellow-Vivid, Teal | Neutral, versatile |
| 6 | Red + Yellow-Vivid | Warm Grey | Cyan, Lime-Green | Warm, energetic |
| 7 | Cyan | Warm Grey | Blue, Red, Yellow, Teal | Approachable, organic |
| 8 | Blue-Vivid | Cool Grey | Cyan-Vivid, Orange-Vivid, Red-Vivid, Yellow-Vivid | Bold, vibrant |
| 9 | Light-Blue-Vivid | Cool Grey | Pink-Vivid, Red-Vivid, Yellow-Vivid, Teal | Friendly, open |
| 10 | Indigo | Cool Grey | Light-Blue-Vivid, Red-Vivid, Yellow-Vivid, Teal | Deep, authoritative |
| 11 | Pink-Vivid | Cool Grey | Purple-Vivid, Cyan-Vivid, Red-Vivid, Yellow-Vivid | Playful, creative |
| 12 | Green | Grey | Purple, Red, Yellow | Natural, grounded |
| 13 | Yellow-Vivid + Light-Blue-Vivid | Grey | Red-Vivid, Teal | Cheerful, dual-accent |
| 14 | Orange + Lime-Green | Grey | Light-Blue, Red, Yellow | Lively, organic |
| 15 | Blue | Blue-Grey | Teal-Vivid, Red, Yellow | Reliable, classic |
| 16 | Purple + Red-Vivid | Blue-Grey | Teal-Vivid, Yellow-Vivid | Dramatic, high-energy |
| 17 | Magenta + Orange-Vivid | Blue-Grey | Yellow-Vivid, Red-Vivid, Green-Vivid | Bold, expressive |
| 18 | Purple | Warm Grey | Cyan, Red-Vivid, Yellow, Green-Vivid | Refined, elegant |
| 19 | Indigo + Orange-Vivid | Cool Grey | Magenta-Vivid, Red-Vivid, Yellow-Vivid, Green-Vivid | Striking contrast |
| 20 | Light-Blue + Green | Cool Grey | Purple, Red, Yellow | Calm, balanced |
| 21 | Orange-Vivid | Cool Grey | Indigo, Red, Yellow, Green | Warm, confident |
| 22 | Indigo + Cyan-Vivid | Cool Grey | Pink-Vivid, Red-Vivid, Yellow-Vivid, Green-Vivid | Tech-forward |
| 23 | Teal-Vivid | Grey | Yellow-Vivid, Red-Vivid | Minimal, focused |
| 24 | Yellow | Grey | Blue, Orange, Red, Green | Sunny, approachable |

## Neutral Families

Each palette uses one of four grey families. The neutral choice has a big
effect on overall feel:

- **Blue-Grey**: Palettes 1-5, 15-17. Professional, cool, tech-friendly.
  Use when: SaaS dashboards, corporate tools, dev tools.
- **Cool Grey**: Palettes 8-11, 19-22. Crisp and modern.
  Use when: Consumer apps, ecommerce, media.
- **Warm Grey**: Palettes 6-7, 18. Friendly, human, organic.
  Use when: Lifestyle brands, blogs, food/wellness.
- **Pure Grey**: Palettes 12-14, 23-24. Fully neutral, doesn't compete.
  Use when: Content-heavy sites, portfolios, minimal design.

## Recommended Starter Palettes

**For most web apps:** Palette 2 (Blue primary, Blue-Grey neutrals).
Blue is the safest choice — universally trusted, works for almost anything.

**For something modern/fresh:** Palette 8 (Blue-Vivid primary, Cool Grey).
Higher saturation gives a more vibrant feel.

**For creative/design tools:** Palette 3 (Purple) or 11 (Pink-Vivid).
Stand out from the sea of blue SaaS products.

**For nature/health:** Palette 12 (Green) or 4 (Teal).
Green/teal conveys growth, health, calm.

**For ecommerce/marketplace:** Palette 21 (Orange-Vivid) or 6 (Red + Yellow).
Warm colors drive action and urgency.

## Assembling Your Palette

Once you pick a palette number, look up the exact hex values in
`color-swatches.md`. You'll need:

1. **Primary**: All 10 shades (050-900)
2. **Neutral/Grey**: All 10 shades
3. **Red/Danger**: Shades 300, 500, 600, 700 (errors, destructive actions)
4. **Yellow/Warning**: Shades 300, 500, 600 (alerts, caution)
5. **Green/Success**: Shades 300, 500, 600 (confirmation, positive states)
6. **1-2 accent colors**: Shades 200, 400, 500, 600 (badges, highlights)

For a minimal setup, you can get by with just primary + neutral + red.
Add more as the design grows.
