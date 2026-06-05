# Huxley Design Principles for {{project_name}}

This file is Huxley's standing memory across sessions. It is the anti-circling mechanism.

## How to read this file

- Each principle has a slug in `[brackets]`. Cite the slug in every finding.
- Principles are dated. Older principles are still active unless explicitly superseded.
- A successor principle starts with `Supersedes [old-slug]: <why>` and is dated. The old principle is NOT deleted — the trail matters.
- Principles are opinionated, short, and product-specific. Universal UX wisdom that does not change how {{project_name}} is built does not belong here.

## How Huxley uses this file

1. Read in full at the start of every session.
2. Cite at least one principle in every finding.
3. If a proposal contradicts a principle, file a `## Principle override` block in the finding AND queue a principles-file update at session end. The bar for overrides is high.
4. At session end, append new principles or sharpen existing ones, dated.

## Principles

### [hierarchy-matches-importance] ({{seed_date}})

Visual weight (size, weight, contrast, position) follows information priority. The thing the user came for is the visual focal point; supporting content sits clearly subordinate; chrome stays out of the way. If multiple elements compete for primary attention on the same surface, the surface fails this principle.

**How to apply:** every page audit checks "what is the primary action / piece of information here?" — and then "is it visually the strongest thing on the screen?". If the answer is no, file a hierarchy finding.

### [consistency-across-surfaces] ({{seed_date}})

The same affordance looks and behaves the same way wherever it appears. Two "Save" buttons should not differ in shape, colour, position, or label across two surfaces. Two empty states for the same kind of list should follow the same pattern. Two "are you sure" confirmations should use the same component.

**How to apply:** when you encounter the second instance of an affordance, compare it to the first. Mismatches are `uxKind: inconsistency` findings.

### [empty-states-are-progress-prompts] ({{seed_date}})

Every empty state offers a concrete next step the user can take from the empty state itself. "No items yet" is not enough. "No workers yet — create your first worker" with a primary button is the minimum standard.

**How to apply:** every list / table / panel that can be empty is audited for its empty state. Empty states without an action are findings.

### [errors-are-actionable] ({{seed_date}})

Every error message tells the user what happened AND what to do next. Raw server errors, technical exception text, and generic "Something went wrong" without a recovery path all fail this principle.

**How to apply:** every error surface (toast, inline form error, error page, notification) is audited. Errors that do not point at a next step are findings.

### [respect-user-time] ({{seed_date}})

The product minimises clicks, scrolls, waits, and cognitive load on high-traffic paths. Defaults should be the most common choice. Sensible inferred values should be inferred. Steps that exist only because the dev-team-found-it-easier-to-build-that-way should be combined or removed.

**How to apply:** for every multi-step journey, count the steps. For every step, ask whether it could be defaulted, inferred, or deferred. Excess steps on high-traffic paths are friction findings.

### [content-before-chrome] ({{seed_date}})

The user's content gets the prime real estate. Navigation, branding, and chrome stay supportive. On a detail page, the user's content is the focal point — not the sidebar, not the top bar, not the breadcrumb. The chrome surfaces information the user needs to act (where am I, where do I go) and otherwise stays quiet.

**How to apply:** on every detail surface, measure how much of the visible viewport is the actual content vs the chrome. If chrome dominates, that is a hierarchy finding.

### [scannable-density-over-spacious] ({{seed_date}})

{{project_name}} is a {{product_type}}. Users scan many items per session. Default to dense over spacious so more is visible per screenful — but never sacrifice readability. Reach for spacious only when the surface is single-purpose (e.g. an empty-state, a confirmation modal).

**How to apply:** list / grid surfaces are audited for density vs spacing. Surfaces that hide most content below the fold without earning the whitespace are findings.

> NOTE for operator: this principle makes a product-type claim. If your product is content-focused (a reader, a marketing site, a creative tool) rather than density-focused (a productivity tool, an admin dashboard, a CRM), REVERSE this principle: prefer spacious by default, dense only where users explicitly scan lists. Edit the principle to fit; do not leave the seed verbatim if it does not describe your product.

### [first-mile-must-feel-effortless] ({{seed_date}})

The first time a new user uses {{project_name}} — from registration through the first value-delivered moment — is the highest-leverage UX surface in the product. Every friction on the first-mile path costs disproportionately because new users have no committed time or sunk-cost reason to push through.

**How to apply:** Huxley walks the first-mile journey end-to-end in every session and audits each step independently of how it feels to a returning user. A papercut on day 50 is a minor finding; the same papercut on day 0 is major.

---

## Sessions

(Append a one-line entry per session: date, scope walked, principles added/sharpened/superseded.)

- {{seed_date}} — Initial seed (no walk yet); 8 starting principles authored by operator at file creation.

---

## Template completion checklist — REMOVE BEFORE USE

When you clone this template into a new project, work through this list and delete the section at the end.

### 1. Replace placeholders

- `{{project_name}}` → your project's display name (appears in 4 places).
- `{{seed_date}}` → today's date in `YYYY-MM-DD` format (appears next to every seed principle's slug).
- `{{product_type}}` → a short noun phrase describing your product (e.g. "productivity tool", "admin dashboard", "CRM", "content reader"). Used in `[scannable-density-over-spacious]`.

### 2. Review the seed principles

- 7 of 8 seed principles are universal — keep them as-is unless your product genuinely contradicts one.
- `[scannable-density-over-spacious]` makes a product-type claim. The inline `NOTE for operator:` block guides whether to keep, reverse, or drop it. Either way, **remove the NOTE block** before checking in (it is a template artefact, not a principle).

### 3. Add any product-specific principles you already know

- If you ship a product with a distinctive design opinion (e.g. "no modals", "every destructive action requires typed confirmation", "we use exactly two type sizes"), add it as a principle now. Huxley will respect it from session 1.
- Keep additions short and slug-prefixed; date them with your seed date.

### 4. Delete this checklist

Delete this entire `## Template completion checklist — REMOVE BEFORE USE` section before checking the file into your new project.
