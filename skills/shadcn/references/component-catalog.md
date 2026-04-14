# shadcn/ui Component Catalog

Quick reference for the most commonly used components and their shadcn patterns. For each component, the source of truth is https://ui.shadcn.com/docs/components — consult it when you need exact class strings or advanced usage.

## Table of Contents

1. [Layout Components](#layout-components)
2. [Form Components](#form-components)
3. [Feedback Components](#feedback-components)
4. [Overlay Components](#overlay-components)
5. [Navigation Components](#navigation-components)
6. [Data Display Components](#data-display-components)

---

## Layout Components

### Card
Pattern: Compound component (no Radix dependency)
Sub-components: `Card`, `CardHeader`, `CardTitle`, `CardDescription`, `CardAction`, `CardContent`, `CardFooter`
Root classes: `rounded-xl border bg-card text-card-foreground shadow-sm`
CardHeader uses CSS Grid with `has-data-[slot=card-action]:grid-cols-[1fr_auto]` for optional action buttons.

### Separator
Pattern: Radix wrapper
Renders a styled `<hr>` with `bg-border` color.
Supports `orientation="horizontal"` (default) and `orientation="vertical"`.

### Accordion
Pattern: Radix wrapper (compound)
Sub-components: `Accordion`, `AccordionItem`, `AccordionTrigger`, `AccordionContent`
Uses `data-[state=open]` for chevron rotation animation.

### Tabs
Pattern: Radix wrapper (compound)
Sub-components: `Tabs`, `TabsList`, `TabsTrigger`, `TabsContent`
TabsList: `inline-flex items-center justify-center rounded-lg bg-muted p-1`
TabsTrigger: `data-[state=active]:bg-background data-[state=active]:shadow-sm`

---

## Form Components

### Input
Pattern: Simple styled element
Classes: `flex h-9 w-full rounded-md border border-input bg-transparent px-3 py-1 text-sm shadow-sm transition-colors placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring disabled:cursor-not-allowed disabled:opacity-50`

### Textarea
Pattern: Simple styled element
Same border/focus pattern as Input, with `min-h-[60px]`.

### Label
Pattern: Radix wrapper
Classes: `text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70`

### Select
Pattern: Radix wrapper (compound)
Sub-components: `Select`, `SelectTrigger`, `SelectValue`, `SelectContent`, `SelectItem`, `SelectGroup`, `SelectLabel`, `SelectSeparator`
SelectTrigger: `flex h-9 items-center justify-between rounded-md border border-input bg-transparent px-3`
SelectContent: Positioned with Radix popper, `bg-popover text-popover-foreground rounded-md border shadow-md`

### Checkbox
Pattern: Radix wrapper
Classes: `h-4 w-4 rounded-sm border border-primary shadow focus-visible:outline-none focus-visible:ring-1 data-[state=checked]:bg-primary data-[state=checked]:text-primary-foreground`

### Switch
Pattern: Radix wrapper
Track: `h-5 w-9 rounded-full border-2 border-transparent bg-input data-[state=checked]:bg-primary`
Thumb: `block h-4 w-4 rounded-full bg-background shadow-lg transition-transform data-[state=checked]:translate-x-4`

### RadioGroup
Pattern: Radix wrapper (compound)
Sub-components: `RadioGroup`, `RadioGroupItem`
Items: `h-4 w-4 rounded-full border border-primary shadow focus-visible:ring-1`

### Slider
Pattern: Radix wrapper
Uses Track, Range, and Thumb sub-components with semantic colors.

---

## Feedback Components

### Alert
Pattern: Variant component (CVA)
Variants: `default` (border bg-background), `destructive` (border-destructive/50 text-destructive)
Sub-components: `Alert`, `AlertTitle`, `AlertDescription`

### Badge
Pattern: Variant component (CVA)
Variants: `default`, `secondary`, `destructive`, `outline`
Base: `inline-flex items-center rounded-md border px-2.5 py-0.5 text-xs font-semibold transition-colors`

### Toast / Sonner
Pattern: Third-party integration
Use the `sonner` library with shadcn styling applied via `toaster` component.

### Progress
Pattern: Radix wrapper
Track: `relative h-2 w-full overflow-hidden rounded-full bg-primary/20`
Indicator: `h-full bg-primary transition-all`

### Skeleton
Pattern: Simple styled element
Classes: `animate-pulse rounded-md bg-primary/10`
Use to show loading states with the shape of the content that will appear.

---

## Overlay Components

### Dialog
Pattern: Radix wrapper
See Pattern C in SKILL.md for full implementation.
Overlay: `fixed inset-0 z-50 bg-black/50`
Content: Centered with translate, `max-w-lg`, includes close button.
Footer: `flex-col-reverse sm:flex-row sm:justify-end` for responsive button layout.

### Sheet
Pattern: Radix wrapper (Dialog variant)
Slides in from a specified side (`top`, `right`, `bottom`, `left`).
Uses `data-[state=open]:animate-in` with slide transitions.

### Popover
Pattern: Radix wrapper
Content: `z-50 w-72 rounded-md border bg-popover p-4 text-popover-foreground shadow-md`
Positioned via Radix popper with configurable `side`, `align`, `sideOffset`.

### Tooltip
Pattern: Radix wrapper
Content: `z-50 rounded-md bg-primary px-3 py-1.5 text-xs text-primary-foreground animate-in`
Always wrap interactive elements with `TooltipTrigger` + `asChild`.

### DropdownMenu
Pattern: Radix wrapper (compound)
Sub-components: `DropdownMenu`, `DropdownMenuTrigger`, `DropdownMenuContent`, `DropdownMenuItem`, `DropdownMenuCheckboxItem`, `DropdownMenuRadioItem`, `DropdownMenuLabel`, `DropdownMenuSeparator`, `DropdownMenuGroup`, `DropdownMenuSub`
Items: `relative flex cursor-default select-none items-center rounded-sm px-2 py-1.5 text-sm outline-none focus:bg-accent focus:text-accent-foreground`

### Command (cmdk)
Pattern: Third-party wrapper (cmdk library)
Used for command palettes and searchable lists.
Sub-components: `Command`, `CommandInput`, `CommandList`, `CommandEmpty`, `CommandGroup`, `CommandItem`, `CommandSeparator`

---

## Navigation Components

### NavigationMenu
Pattern: Radix wrapper (compound)
For top-level site navigation with optional mega-menu dropdowns.

### Breadcrumb
Pattern: Compound component (no Radix)
Sub-components: `Breadcrumb`, `BreadcrumbList`, `BreadcrumbItem`, `BreadcrumbLink`, `BreadcrumbPage`, `BreadcrumbSeparator`, `BreadcrumbEllipsis`

### Pagination
Pattern: Compound component (no Radix)
Uses `buttonVariants` for consistent styling with Button.

### Sidebar
Pattern: Compound component with context
Uses `--sidebar-*` color tokens for independent theming.
Sub-components: `SidebarProvider`, `Sidebar`, `SidebarHeader`, `SidebarContent`, `SidebarGroup`, `SidebarMenu`, `SidebarMenuItem`, `SidebarMenuButton`, `SidebarFooter`, `SidebarTrigger`

---

## Data Display Components

### Table
Pattern: Compound component (no Radix)
Sub-components: `Table`, `TableHeader`, `TableBody`, `TableFooter`, `TableHead`, `TableRow`, `TableCell`, `TableCaption`
Root: `w-full caption-bottom text-sm`
Row: `border-b transition-colors hover:bg-muted/50 data-[state=selected]:bg-muted`

### Avatar
Pattern: Radix wrapper
Sub-components: `Avatar`, `AvatarImage`, `AvatarFallback`
Root: `relative h-10 w-10 shrink-0 overflow-hidden rounded-full`
Fallback: `flex h-full w-full items-center justify-center rounded-full bg-muted`

### HoverCard
Pattern: Radix wrapper
Content: `z-50 w-64 rounded-md border bg-popover p-4 text-popover-foreground shadow-md`

### Calendar
Pattern: Third-party (react-day-picker) with shadcn styling
Uses `buttonVariants` for day cells, semantic tokens for selected/today states.

### Chart
Pattern: Third-party (Recharts) with shadcn config
Uses `--chart-1` through `--chart-5` color tokens.
Wrap with `ChartContainer` for responsive sizing and theme integration.
