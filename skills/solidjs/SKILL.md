---
name: solidjs
description: >
  SolidJS frontend development skill. Produces clean, readable, idiomatic SolidJS
  code using control-flow components, reactive primitives, and proper component
  patterns. Avoids code repetition and reuses existing project code.
  TRIGGER when: code imports from "solid-js", "solid-js/web", "solid-js/store",
  "@solidjs/router", "@solidjs/meta", or "solid-start"; user asks to build UI
  with SolidJS or Solid; project has SolidJS dependencies in package.json; user
  mentions Solid signals, stores, createEffect, createResource, or Solid
  components. Use this skill even if the user just says "Solid" without "JS".
  DO NOT TRIGGER when: code uses React, Vue, Svelte, Angular, or other frameworks.
---

# SolidJS Frontend Development

Write clean, readable, idiomatic SolidJS code. Prioritize code reuse, clarity,
and the SolidJS reactive model.

**Related skills:** Combine with `ui` for visual design principles (spacing
scales, color palettes, typography, hierarchy) and `shadcn` for component
structure and composition patterns — both are framework-agnostic and apply
directly to SolidJS projects.

## Before Writing Code

1. **Read existing code first.** Search the project for existing components,
   utilities, stores, and styles before creating anything new. Reuse what exists.
2. **Understand the reactive graph.** Identify which values are signals, which are
   derived, and which are side effects. Get the mental model right before coding.
3. **Check for shared patterns.** Look at how the project handles routing, data
   fetching, error states, and layouts. Follow established conventions.

## Control-Flow Components

SolidJS control-flow components exist because Solid's rendering is not a
re-render-the-whole-tree model like React — JSX runs once and only reactive
expressions update. Using `<Show>`, `<For>`, and `<Switch>` ensures that Solid
can granularly track which DOM nodes to create, update, or dispose. Inline
ternaries and `.map()` in JSX bypass this, causing unnecessary DOM destruction
and recreation.

Use control-flow components for **rendering conditional or repeated JSX**.
Ternaries and logical expressions are fine for **prop values and non-JSX logic**.

### `<Show>` for conditional rendering

```tsx
// Avoid — ternary for conditional JSX blocks
{user() ? <Profile user={user()} /> : null}
{isLoading() && <Spinner />}

// Prefer — <Show> preserves reactivity boundaries
<Show when={user()} fallback={null}>
  {(u) => <Profile user={u()} />}
</Show>

<Show when={isLoading()}>
  <Spinner />
</Show>
```

Use the callback form `{(item) => ...}` when you need the narrowed non-null value.
Use `fallback` for the else branch.

Use `keyed` when the child should fully re-mount when the `when` value changes
identity (not just truthiness):

```tsx
// Without keyed: child updates reactively when selectedUser() changes fields
<Show when={selectedUser()}>
  {(user) => <UserDetail user={user()} />}
</Show>

// With keyed: child unmounts and remounts when selectedUser() is a different object
<Show when={selectedUser()} keyed>
  {(user) => <UserDetail user={user} />}
</Show>
```

### `<For>` for lists

```tsx
// Avoid in JSX — bypasses Solid's keyed reconciliation
{items().map((item) => <Card item={item} />)}

// Prefer — <For> tracks each item by reference, only updates changed nodes
<For each={items()}>
  {(item, index) => <Card item={item} position={index()} />}
</For>
```

`<For>` is keyed by reference — it tracks each object and moves DOM nodes when
the array reorders. Use `<Index>` when items are primitives or when the array
index is the stable identity.

Note: `.map()` is perfectly fine for **data transformation** outside of JSX
(e.g., preparing a derived list in a function). The concern is only about
rendering in JSX.

### `<Switch>` / `<Match>` for multi-branch conditions

```tsx
// Avoid — chained ternaries are hard to read and error-prone
{status() === "loading" ? <Spinner /> : status() === "error" ? <Error /> : <Content />}

// Prefer — <Switch> / <Match> is readable and handles fallback cleanly
<Switch fallback={<Content />}>
  <Match when={status() === "loading"}>
    <Spinner />
  </Match>
  <Match when={status() === "error"}>
    <Error />
  </Match>
</Switch>
```

### `<ErrorBoundary>` for error handling

```tsx
<ErrorBoundary fallback={(err, reset) => <ErrorDisplay error={err} onRetry={reset} />}>
  <RiskyComponent />
</ErrorBoundary>
```

### `<Suspense>` for async boundaries

```tsx
<Suspense fallback={<Skeleton />}>
  <AsyncContent />
</Suspense>
```

### `<Portal>` for rendering outside the DOM hierarchy

```tsx
<Portal mount={document.getElementById("modal-root")!}>
  <Modal />
</Portal>
```

### `<Dynamic>` for dynamic element/component selection

```tsx
<Dynamic component={isLink ? "a" : "button"} href={isLink ? url : undefined}>
  {label}
</Dynamic>
```

(Ternaries for prop values like `component={isLink ? "a" : "button"}` are fine —
the concern is about conditional *JSX blocks*, not prop expressions.)

## Reactivity Rules

### Signals and stores

```tsx
// Simple values — createSignal
const [count, setCount] = createSignal(0);

// Complex/nested objects — createStore
const [state, setState] = createStore({ users: [], filter: "" });

// Update nested store values with path syntax
setState("users", 0, "name", "Alice");
setState("users", (prev) => [...prev, newUser]);
```

### Derived values — no `useMemo` needed

```tsx
// Unnecessary for cheap derivations
const doubled = createMemo(() => count() * 2); // overhead not justified

// A plain function is idiomatic for simple derivations
const doubled = () => count() * 2;
```

Use `createMemo` only when the computation is expensive or consumed by multiple
reactive subscribers. For simple derivations, a plain function is correct —
Solid's fine-grained reactivity means it only runs when accessed in a tracking
scope.

### Effects

Solid has three effect primitives. Choose based on timing needs:

```tsx
// createEffect — runs after render, tracks dependencies automatically
createEffect(() => {
  console.log("Count changed:", count());
});

// createRenderEffect — runs before browser paint (useful for DOM measurement)
createRenderEffect(() => {
  ref.style.height = `${computedHeight()}px`;
});

// createComputed — runs synchronously during tracking (upstream updates)
// Use sparingly; mainly for derived state that must be synchronous
createComputed(() => {
  setFullName(`${firstName()} ${lastName()}`);
});
```

Cleanup with `onCleanup`:

```tsx
createEffect(() => {
  const handler = () => { /* ... */ };
  window.addEventListener("resize", handler);
  onCleanup(() => window.removeEventListener("resize", handler));
});
```

Signals read inside `setTimeout`, `setInterval`, or event handlers are **not
tracked** because those callbacks run outside Solid's reactive context. Read
the value before the async boundary or use `createEffect` to react to changes.

### Explicit dependency tracking with `on()`

When you want an effect to react to specific signals without tracking others
it reads, use the `on()` helper:

```tsx
// Only re-runs when userId() changes, reads config() without tracking it
createEffect(on(userId, (id) => {
  fetchUser(id, config());
}));

// Defer: skip the initial run
createEffect(on(userId, (id) => {
  refetch(id);
}, { defer: true }));
```

### Batching multiple updates

When updating multiple signals in sequence, Solid notifies subscribers after
each one. Wrap related updates in `batch()` to defer notifications until all
updates are applied:

```tsx
batch(() => {
  setFirstName("Alice");
  setLastName("Smith");
  setAge(30);
});
// Dependents re-compute once, not three times
```

### Resources for async data

```tsx
const [data] = createResource(userId, fetchUser);

<Show when={!data.loading} fallback={<Spinner />}>
  <Profile user={data()!} />
</Show>
```

Prefer `createResource` over manual signal + effect patterns for data fetching.
It integrates with `<Suspense>` and handles loading/error states.

## Context API

Use `createContext` / `useContext` for shared state that multiple components
need without prop drilling. Context is the idiomatic way to do dependency
injection in SolidJS:

```tsx
// Create the context with a default value
const ThemeContext = createContext<{ theme: Accessor<string>; toggle: () => void }>();

// Provider component
export function ThemeProvider(props: ParentProps) {
  const [theme, setTheme] = createSignal("light");
  const toggle = () => setTheme((t) => (t === "light" ? "dark" : "light"));

  return (
    <ThemeContext.Provider value={{ theme, toggle }}>
      {props.children}
    </ThemeContext.Provider>
  );
}

// Consumer hook — wraps useContext for safety
export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error("useTheme must be used within ThemeProvider");
  return ctx;
}
```

Prefer context over global stores when the state is scoped to a subtree.
Use global stores (`createStore` at module level) when the state is truly
app-wide.

## Component Patterns

### Props

SolidJS props are a reactive proxy. Destructuring breaks the proxy and loses
reactivity — signals stop updating. Access props through the object or use
`splitProps`/`mergeProps`:

```tsx
// Destructuring breaks reactivity — the values become static snapshots
const MyComponent = ({ name, count }) => { ... }; // broken

// Access props directly — reactivity preserved
const MyComponent = (props: { name: string; count: number }) => {
  return <div>{props.name}: {props.count}</div>;
};

// splitProps when forwarding rest props to a child element
const MyComponent = (props) => {
  const [local, rest] = splitProps(props, ["class", "onClick"]);
  return <button class={local.class} onClick={local.onClick} {...rest} />;
};
```

### Children

```tsx
// Resolve children reactively
const MyWrapper = (props: ParentProps) => {
  const resolved = children(() => props.children);
  return <div class="wrapper">{resolved()}</div>;
};
```

Use the `children()` helper when you need to transform or inspect children.

### Component composition over config props

```tsx
// Monolithic — hard to customize, test, or extend
<DataTable sortable filterable paginated exportable columns={...} />

// Composable — each piece is independent and reusable
<DataTable data={items()}>
  <DataTable.Header>
    <DataTable.Sort field="name" />
    <DataTable.Filter />
  </DataTable.Header>
  <DataTable.Body />
  <DataTable.Pagination />
</DataTable>
```

## Routing with @solidjs/router

When the project uses `@solidjs/router`, follow these patterns:

```tsx
import { Router, Route, A, useParams, useNavigate, useSearchParams } from "@solidjs/router";

// Route definitions
<Router>
  <Route path="/" component={Home} />
  <Route path="/users/:id" component={UserDetail} />
  <Route path="*404" component={NotFound} />
</Router>

// Link component — use <A> instead of <a> for client-side navigation
<A href="/users/42" activeClass="active">Profile</A>

// Read route params reactively
const UserDetail = () => {
  const params = useParams<{ id: string }>();
  const [user] = createResource(() => params.id, fetchUser);
  // params.id is reactive — resource refetches when the route changes
  return <Show when={user()}>{(u) => <Profile user={u()} />}</Show>;
};

// Programmatic navigation
const navigate = useNavigate();
navigate("/dashboard", { replace: true });

// Search params (query strings)
const [searchParams, setSearchParams] = useSearchParams();
setSearchParams({ page: "2" });
```

## Code Reuse

### Extract shared logic into hooks (functions returning reactive primitives)

```tsx
function useClickOutside(ref: () => HTMLElement | undefined, callback: () => void) {
  createEffect(() => {
    const el = ref();
    if (!el) return;
    const handler = (e: MouseEvent) => {
      if (!el.contains(e.target as Node)) callback();
    };
    document.addEventListener("mousedown", handler);
    onCleanup(() => document.removeEventListener("mousedown", handler));
  });
}
```

### Reuse existing components — search before creating

Before creating a new component, search the codebase:
- `Glob` for `**/*.tsx` and `**/*.jsx` files
- `Grep` for component names, similar UI patterns, shared utilities

If a similar component exists, extend or compose it instead of duplicating.

### Shared styles

Follow the project's existing CSS approach (CSS modules, Tailwind, vanilla-extract,
etc.). Do not introduce a different styling method.

## Readability Standards

1. **One component per file.** Small helpers used only by that component can stay.
2. **Descriptive names.** `UserProfileCard`, not `UPC`. `handleSubmit`, not `hs`.
3. **Flat JSX.** Avoid deep nesting beyond 3-4 levels. Extract sub-components.
4. **Group related logic.** Signals, derived values, and their effects should be
   adjacent, not scattered across the component body.
5. **Early returns** for guard clauses and error states.
6. **No magic numbers or strings.** Use named constants.

## Common Mistakes to Avoid

| Mistake | Why it breaks | Fix |
|---------|--------------|-----|
| Destructuring props | Breaks the reactive proxy — values become static | Access `props.x` directly |
| Calling signals without `()` | Returns the getter function, not the value | Always call: `count()` |
| `.map()` for rendering lists | Bypasses Solid's keyed reconciliation, re-creates DOM | Use `<For>` or `<Index>` |
| Ternaries for conditional JSX blocks | Harder to read, loses reactive disposal boundaries | Use `<Show>` / `<Switch>` |
| `createMemo` for cheap ops | Adds a memo node for no benefit | Use plain derived functions |
| Multiple signal updates without `batch` | Each update triggers subscribers separately | Wrap in `batch()` |
| `async` in `createEffect` | Effect stops tracking after the first `await` | Split into sync tracking + `createResource` |
| Forgetting `onCleanup` in effects | Listeners/subscriptions leak across re-runs | Add `onCleanup` in the same effect |
| Signals in `setTimeout`/handlers | Outside reactive context, not tracked | Read value before async boundary |

## Project Structure Conventions

Follow whatever structure the project already uses. If starting fresh, prefer:

```
src/
├── components/          # Reusable UI components
│   ├── Button.tsx
│   └── Modal.tsx
├── features/            # Feature-specific components and logic
│   └── auth/
│       ├── LoginForm.tsx
│       └── useAuth.ts
├── stores/              # Global state (createStore / context)
├── api/                 # API client and types
├── utils/               # Pure utility functions
└── App.tsx
```
