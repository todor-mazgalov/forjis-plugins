---
name: shadcn
description: >
  Enforces shadcn/ui design principles and component patterns for web UI,
  framework- and styling-agnostic. Covers component ownership (local source
  under components/ui/), accessible primitives, dark mode, forms, dialogs,
  cards, and composable layout patterns. Styling approach is flexible —
  Tailwind, plain CSS, SCSS, CSS Modules, or other styling systems are all
  viable; the skill focuses on structural and design conventions, not the
  styling layer. Applies equally to React, SolidJS, Vue, Svelte, and any
  web framework that can adopt the same composition conventions.
  TRIGGER when: project is a web frontend (any framework, any styling system),
  or the user asks to build, style, or refactor web components, pages, forms,
  dialogs, cards, or any web UI element where a consistent design system
  applies. Also trigger when creating new web pages or layouts even if
  shadcn is not explicitly mentioned.
  DO NOT TRIGGER when: project is a React Native app (use react-native), a
  Salesforce LWC project (use sf-lwc), a native mobile app (Android/iOS), or
  any non-web target.
---

# shadcn/ui Design System Skill

This skill ensures all components and pages follow the design principles of [shadcn/ui](https://ui.shadcn.com/docs/components). The *principles* are framework- and styling-agnostic — they apply whether you're writing React + Tailwind, SolidJS + CSS Modules, Vue + SCSS, or Svelte + plain CSS. The *code examples* in this document are written in React + Tailwind because that's the canonical shadcn implementation, but the structural and design conventions translate directly to any web stack.

**Related skills:** This skill covers component structure and composition. For the underlying visual design principles (spacing scales, color theory, typographic hierarchy, shadows), combine it with the `ui` skill which encodes Refactoring UI principles. Framework-specific idioms come from the corresponding framework skill (`solidjs`, etc.).

When working in a non-React / non-Tailwind project, adapt the examples to the project's existing framework and styling system. Do **not** introduce React, Tailwind, Radix, or `class-variance-authority` into a project that doesn't already use them.

## Core Philosophy

shadcn/ui is built on five principles. Internalize these — they inform every decision regardless of stack.

1. **Open Code / Ownership** — Components live in your codebase (typically `components/ui/`). You own them, modify them directly, and never wrap around a locked library API. Write them as local source files, not imports from a package.

2. **Composition over Configuration** — Build complex UI from small, composable sub-components rather than monolithic components with dozens of props. A `Card` is composed of `CardHeader`, `CardTitle`, `CardDescription`, `CardContent`, `CardFooter` — not a single `<Card title="..." description="..." />`.

3. **Beautiful Defaults** — Components should look professional out of the box. Use semantic color tokens, consistent spacing, and a unified radius scale so everything feels cohesive without manual tweaking.

4. **Accessibility First** — Use accessible primitives as the behavioral foundation for interactive components (Dialog, Popover, Select, Dropdown, etc.). In React, Radix UI is the canonical choice; in other ecosystems use the equivalent (e.g., Kobalte for SolidJS, Radix Vue / Reka UI for Vue, Melt UI / Bits UI for Svelte). Whatever you pick must provide focus trapping, keyboard navigation, ARIA attributes, and screen reader support.

5. **Pluggable Styling Layer** — Styling is orthogonal to the design system. Tailwind CSS is the canonical choice, but plain CSS, SCSS, CSS Modules, vanilla-extract, or linaria are all viable. Pick one and be consistent inside a project. Don't mix multiple styling systems in the same component.

## Class Merging Helper (stack-dependent)

If the project uses Tailwind, every component should merge incoming class names with defaults so consumers can override without fighting specificity:

```typescript
// React + Tailwind canonical helper
import { clsx, type ClassValue } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

For other styling systems, the equivalent is:

- **Plain CSS / SCSS / CSS Modules** — accept a `class` / `className` prop and concatenate it with the default class string. No tailwind-merge is needed because class specificity works normally.
- **SolidJS** — same `clsx`/`twMerge` pattern, but accept `class` instead of `className`.
- **Vue** — use the framework's `:class` binding which already merges arrays/objects; no helper needed.

The rule that matters: **every component that accepts a class name prop must merge it with defaults**, regardless of how.

## Component Patterns

There are three fundamental patterns. Use the right one for the job. The code below is React + Tailwind + Radix; translate to your stack.

### Pattern A: Variant Components (e.g., Button)

For components with multiple visual states, use a variants helper. In React + Tailwind, `class-variance-authority` (CVA) is the canonical choice:

```tsx
import { cva, type VariantProps } from "class-variance-authority"
import { Slot } from "radix-ui"
import { cn } from "@/lib/utils"

const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline: "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-9 px-4 py-2",
        sm: "h-8 rounded-md px-3 text-xs",
        lg: "h-10 rounded-md px-6",
        icon: "h-9 w-9",
      },
    },
    defaultVariants: { variant: "default", size: "default" },
  }
)

function Button({ className, variant, size, asChild = false, ...props }:
  React.ComponentProps<"button"> & VariantProps<typeof buttonVariants> & { asChild?: boolean }
) {
  const Comp = asChild ? Slot.Root : "button"
  return <Comp className={cn(buttonVariants({ variant, size, className }))} {...props} />
}

export { Button, buttonVariants }
```

Key points (stack-agnostic):
- Centralize variant definitions so they can be reused and kept consistent
- Provide a way to compose with other elements (React/Vue/Solid all have a slot-style pattern)
- Always define default variants
- In plain CSS, this becomes a BEM-style modifier class system (`.button`, `.button--destructive`, `.button--sm`). In CSS Modules, export variant class maps and pick by key. In SCSS, use `@include` mixins per variant.

### Pattern B: Compound Components (e.g., Card)

For structural components, create named sub-components:

```tsx
function Card({ className, ...props }: React.ComponentProps<"div">) {
  return <div data-slot="card" className={cn("rounded-xl border bg-card text-card-foreground shadow-sm", className)} {...props} />
}

function CardHeader({ className, ...props }: React.ComponentProps<"div">) {
  return <div data-slot="card-header" className={cn("flex flex-col gap-1.5 p-6", className)} {...props} />
}

function CardTitle({ className, ...props }: React.ComponentProps<"div">) {
  return <div data-slot="card-title" className={cn("leading-none font-semibold", className)} {...props} />
}

function CardDescription({ className, ...props }: React.ComponentProps<"div">) {
  return <div data-slot="card-description" className={cn("text-muted-foreground text-sm", className)} {...props} />
}

function CardContent({ className, ...props }: React.ComponentProps<"div">) {
  return <div data-slot="card-content" className={cn("p-6 pt-0", className)} {...props} />
}

function CardFooter({ className, ...props }: React.ComponentProps<"div">) {
  return <div data-slot="card-footer" className={cn("flex items-center p-6 pt-0", className)} {...props} />
}
```

Key points (stack-agnostic):
- Use `data-slot` attributes for CSS targeting and for styling hooks that work across any styling system
- Every sub-component accepts and merges the class name prop
- Spread remaining props so consumers can pass any HTML attribute
- In SolidJS, replace `React.ComponentProps<"div">` with `JSX.IntrinsicElements["div"]` and use `class` instead of `className`
- In Vue, use scoped slots + a root `<div>` with `v-bind="$attrs"`
- In Svelte, use `<svelte:element>` or a root `<div>` with `{...$$restProps}`

### Pattern C: Accessible Primitive Wrappers (e.g., Dialog)

For interactive components, wrap an accessible primitive library with the project's styling:

```tsx
import * as DialogPrimitive from "radix-ui/dialog"

function DialogOverlay({ className, ...props }: React.ComponentProps<typeof DialogPrimitive.Overlay>) {
  return (
    <DialogPrimitive.Overlay
      className={cn("fixed inset-0 z-50 bg-black/50 data-[state=open]:animate-in data-[state=closed]:animate-out", className)}
      {...props}
    />
  )
}

function DialogContent({ className, children, ...props }: React.ComponentProps<typeof DialogPrimitive.Content>) {
  return (
    <DialogPrimitive.Portal>
      <DialogOverlay />
      <DialogPrimitive.Content
        className={cn(
          "fixed top-[50%] left-[50%] z-50 w-full max-w-lg translate-x-[-50%] translate-y-[-50%] rounded-lg border bg-background p-6 shadow-lg",
          className
        )}
        {...props}
      >
        {children}
        <DialogPrimitive.Close className="absolute top-4 right-4 rounded-sm opacity-70 hover:opacity-100">
          <XIcon className="h-4 w-4" />
        </DialogPrimitive.Close>
      </DialogPrimitive.Content>
    </DialogPrimitive.Portal>
  )
}
```

Key points (stack-agnostic):
- The primitive library provides behavior + accessibility, you provide styling
- Use state-driven attributes (`data-[state=open/closed]` in the examples) for styling and animations — every accessible primitive library exposes similar attributes
- Overlays layer above page content; centered content uses a translate pattern
- Always include accessible close affordances
- Equivalents by framework: Radix UI (React), Kobalte (SolidJS), Radix Vue / Reka UI (Vue), Melt UI / Bits UI (Svelte)

## Theming: Semantic Color Tokens

Never use raw colors. Always use semantic tokens via CSS custom properties. This works identically in every framework and every styling system — CSS variables are universal.

| Token | Purpose |
|-------|---------|
| `--background` / `--foreground` | App background and default text |
| `--card` / `--card-foreground` | Elevated card surfaces |
| `--popover` / `--popover-foreground` | Floating elements |
| `--primary` / `--primary-foreground` | Primary actions and branding |
| `--secondary` / `--secondary-foreground` | Lower-emphasis actions |
| `--muted` / `--muted-foreground` | Subtle surfaces and subdued text |
| `--accent` / `--accent-foreground` | Hover/focus states |
| `--destructive` | Error/danger states |
| `--border` | Borders and separators |
| `--input` | Form control borders |
| `--ring` | Focus rings |
| `--radius` | Base border-radius |

Every surface color has a matching `-foreground` for text on that surface. Reference them via `var(--primary)` and `var(--primary-foreground)` in plain CSS/SCSS, or via the Tailwind token classes (`bg-primary`, `text-primary-foreground`) if using Tailwind. Never use raw color values like `#3b82f6` or `bg-blue-500`.

Colors use OKLCh notation: `oklch(lightness chroma hue)`. Dark mode is toggled via a `.dark` class on the root — same variable names, different values, components respond automatically. This mechanism is independent of framework or styling system.

### Border Radius Scale

Derive all radii from a single `--radius` base:
- `--radius-sm`: 60% of base
- `--radius-md`: 80% of base
- `--radius-lg`: base (100%)
- `--radius-xl`: 140% of base

Use the `lg` radius for cards, `md` for buttons and inputs, `sm` for badges and small elements. Whether you reach these via Tailwind's `rounded-lg` or plain CSS `border-radius: var(--radius-lg)` is a project choice.

## Page Design Principles

When designing pages and layouts, follow these conventions. They're expressed here as Tailwind utility classes because that's the canonical form; translate to your styling system as needed.

### Spacing and Layout
- Use consistent spacing tokens: roughly 24px (`p-6`) for card padding, 16–24px gaps between elements
- Mobile-first responsive design: start with the smallest breakpoint, add larger-screen overrides progressively
- Use CSS Grid or Flexbox for layout composition: e.g., `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6`
- Page containers: cap width (~1280px), center horizontally, add responsive horizontal padding

### Typography
- Headings: bold, tight tracking (`text-3xl font-bold tracking-tight` for page titles), scale down for sub-headings
- Body text: 14–16px with muted foreground for secondary text
- Use tight leading on card titles, relaxed leading on longer prose

### Interactive States
- Focus: visible focus ring using `--ring` token with 3px width and outline offset
- Disabled: reduced opacity (~50%) and no pointer events
- Hover: quick color transitions on interactive elements
- State animations: enter/exit animations driven by `data-state` attributes from primitive libraries

### Forms
- Consistent input height (default ~36px, compact ~32px)
- Labels above inputs with small gap between label and control
- Use destructive color for error messages below inputs
- Group related fields with consistent vertical spacing

### Dark Mode
- Never hardcode colors — semantic tokens handle light/dark automatically
- Test both modes: ensure sufficient contrast in both

## Props Convention

Every component must follow this structural pattern (expressed in React here, adapted per framework):

```tsx
function ComponentName({
  className,
  // ...destructured custom props with defaults
  ...props
}: React.ComponentProps<"element"> & CustomProps) {
  return (
    <element
      data-slot="component-name"
      className={cn("default-classes", className)}
      {...props}
    />
  )
}
```

Rules (stack-agnostic):
- Accept the native element's props so consumers can pass any HTML attribute
- Always accept and merge the class name prop with defaults
- Always forward remaining props to the root element
- Use `data-slot` attributes on sub-components for styling hooks
- Prefer named exports over default exports

**SolidJS adaptation** — use `splitProps(props, ["class"])` and `<element class={...} {...rest}>`; avoid destructuring props directly.

**Vue adaptation** — use `<script setup>`, `defineProps`, and let the root element inherit `$attrs` automatically; forward `class` via `:class`.

**Svelte adaptation** — use `$$restProps` on the root element and accept `class` via `export let class_` (rename because `class` is reserved) or the newer `$props()` rune in Svelte 5.

## Reference

For the full component catalog and live examples, consult `references/component-catalog.md`. For theming details beyond what's covered here, consult `references/theming-guide.md`. Both references use React + Tailwind as the canonical form.
