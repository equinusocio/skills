# React components

Apply whenever creating or editing React components.

## Shape

- Components are always **named `const` arrow functions** (export named or not as needed).

```tsx
const MyComponent = () => (
  // ...
)

export const MyComponent = () => {
  // ...
}
```

- Every component is typed with **`React.FC<>`** when they have props, with props passed to the generic. Otherwise just `React.FC`.

```tsx
import React from 'react'

const MyComponent: React.FC<MyComponentProps> = () => (
  // ...
)
```

## React utility types

- Prefer the **`React.`** namespace for React utility types — `React.FC`, `React.ComponentPropsWithRef`, `React.ComponentPropsWithoutRef`, `React.CSSProperties`, etc.
- Use a single React import (`import React from 'react'` or `import * as React from 'react'`, matching the project). **Do not** named-import those utilities from `'react'` (avoids import clutter).

```tsx
// Prefer
import React from 'react'
const dynamicStyle: React.CSSProperties = {}

// Avoid
import type { CSSProperties, FC, ComponentPropsWithRef } from 'react'
```

## Props type

- Every component has a **`ComponentNameProps`** type. Export it when callers need it outside the module (reuse, inference, wrapping). Otherwise leave it unexported.
- Prefer exporting the **component props type** (`BadgeProps`) over satellite types (variant unions, option aliases, etc.). Consumers take nested pieces via indexed access: `BadgeProps['variant']`.
- When a union (or other alias) appears **only once** on the props type, **inline it** — do not create a separate named type.

```tsx
// Prefer — union once; export only BadgeProps
export type BadgeProps = React.ComponentPropsWithRef<"span"> & {
  /**
   * Visual status treatment.
   * @defaultValue 'neutral'
   */
  variant?: "neutral" | "positive" | "negative";
};

export const Badge: React.FC<BadgeProps> = ({
  children,
  className,
  variant = "neutral",
  ...otherProps
}) => (
  <span {...otherProps} className={clsx(styles.Badge, className)} data-variant={variant}>
    {children}
  </span>
);

// Avoid — unnecessary ChipVariants alias + export
export type ChipVariants = "neutral" | "positive" | "negative";

export type BadgeProps = React.ComponentPropsWithRef<"span"> & {
  variant?: ChipVariants;
};
```

- If a separate alias is still useful inside the file (reused across several props/helpers), keep it **unexported**. Consumers reach it via the props type (`BadgeProps['variant']`), not a second public export.
- Do **not** export helper or file-internal types unless consumers need them as a first-class public API, or they are already surfaced through other exported types (composition, indexed access, `typeof`, etc.). See also type-export guidance in [`style.md`](style.md).
- Prefer props that **extend the HTML (or component) props of the outermost wrapper** — the element/component that receives the props spread.
- Use **`React.ComponentPropsWithRef`** / **`React.ComponentPropsWithoutRef`** as appropriate. In React 19, **`ref` is a normal prop** (no `forwardRef` required for that reason alone).
- Props must always have a TSDoc comment that describes them and an `@defaultValue` marker with the default value assigned to the prop
- When a prop’s type must be inferred from another type or inherited, do not redeclare it — use the original type if you have access. Example:

```tsx
export type MyComponentProps = {
  padding?: StackProps["padding"];
}
```

Or inherit it from the component itself using `typeof`

```tsx
export type MyComponentProps = React.ComponentPropsWithRef<'div'> & {
  /**
   * Panel title shown in the header.
   * @defaultValue 'Panel'
   */
  title?: string
}

export type MyComponentProps = React.ComponentPropsWithRef<typeof OtherComponent> & {
  /**
   * Accent highlight on the card.
   * @defaultValue false
   */
  accent?: boolean
}
```

Use `React.ComponentPropsWithoutRef` when the wrapper must not accept `ref`.

## Destructuring and spread

- Destructure props. Prefer spreading the rest onto the wrapper for props not handled directly.
- Place the spread so it either **preserves defaults** or **lets callers override** — choose deliberately.

```tsx
type MyComponentProps = {
  prop1: string
  prop2?: string
}

const MyComponent: React.FC<MyComponentProps> = ({
  prop1,
  ...otherProps
}) => <div data-prop={prop1} {...otherProps} />
```

## Default values

- Prefer **default values in the parameter list** (including when combined with spread):

```tsx
type MyComponentProps = {
  prop1?: string
  prop2?: string
}

const MyComponent: React.FC<MyComponentProps> = ({
  prop1 = 'default',
  ...otherProps
}) => <div data-prop={prop1} {...otherProps} />
```

## Markup branching

- Prefer **`&&`** when a branch returns `null` (render nothing).
- Prefer a **ternary** when both branches render something — **never nest** ternaries.

```tsx
return (
  <div>
    {condition && <p />}
    {condition2 ? <p /> : <figure />}
  </div>
)
```

## Nullish coalescing

- Prefer **nullish coalescing** (`??`) where possible.

```tsx
const myConst = condition ?? condition2
```

## CSS imports

- CSS modules: import as `styles`.
- Plain CSS: side-effect import (no binding).

```tsx
import styles from './my-component.module.css'

const MyComponent: React.FC = () => <div className={styles.MyClass} />
```

```tsx
import './my-component.css'

const MyComponent: React.FC = () => <div className="MyComponent" />
```

For styling conventions, use the `authoring-css` skill when present. For JS/TS/JSX syntax and lint-style constraints, see [`style.md`](style.md).

## TypeScript path aliases and imports

- When TypeScript path aliases are configured in the project, always use them where applicable.
- When no TypeScript path aliases are configured, recommend that the user configures them.
- Never use deep imports when an exported relative `index` module is available; import from that index instead.

## Prefer project tools

- Avoid custom code or excessive scripting when project tools already cover the need and can shrink the code.

## Performance

- Keep React code performant for re-renders, loading, and data fetching.
- Evaluate when to use `useMemo`, `useCallback`, `React.memo`, `useOptimistic`, `Suspense`, and similar — apply them when they reduce real cost, not by default everywhere. Prioritize performant UX (reactiveness) and optimistic loadings.

## Stay inside React

React owns the UI tree it renders. Prefer React’s model (props, state, refs, JSX events, effects) over escaping to raw DOM APIs on nodes React already manages. Imperative DOM is an **escape hatch**, not the default.

### Forbidden on React-owned DOM

Do **not** use these to find, mutate, or listen to elements that React renders (or should render):

| Escape (avoid) | Prefer |
| --- | --- |
| `document.querySelector` / `querySelectorAll` | `useRef`, callback ref, `ref` prop |
| `getElementById` / `getElementsBy*` / `closest` from globals | Ref to the node (or pass data via props/context) |
| `element.addEventListener` / `removeEventListener` | JSX handlers (`onClick`, `onKeyDown`, …) + named functions in the body |
| `element.classList.add/remove/toggle` | `className` (+ project merge util), or `data-*` + CSS |
| `element.setAttribute` / `removeAttribute` / `element.style.* =` | JSX props, `dynamicStyle` / CSS variables (see below) |
| `element.innerHTML` / `insertAdjacentHTML` | JSX children; `dangerouslySetInnerHTML` only when unavoidable |
| `document.createElement` + `appendChild` / `removeChild` for UI | JSX / conditional render / keys / portals |
| `ReactDOM.render` / `createRoot` into a node React already owns | Compose components; one root owns that subtree |
| Reading the DOM to rediscover state React already has | Props, state, context, derived values |

Also avoid: string refs, `findDOMNode`, `isMounted` (see [`style.md`](style.md)).

```tsx
// Avoid — leaves React’s lifecycle
useEffect(() => {
  document.querySelectorAll('.row').forEach((el) => {
    el.addEventListener('click', onRowClick)
    el.classList.toggle('is-open', isOpen)
  })
}, [isOpen])

// Prefer — stay in React
const panelRef = useRef<HTMLDivElement>(null)

useEffect(() => {
  panelRef.current?.focus()
}, [isOpen])

return (
  <div
    ref={panelRef}
    className={clsx(styles.Panel, isOpen && styles.isOpen)}
    data-open={isOpen ? 'true' : 'false'}
    onClick={handleClick}
  />
)
```

### Correct React approaches

- **UI from data:** render from props/state; re-render updates the DOM. Do not sync “truth” by mutating nodes by hand.
- **Refs:** `useRef` / callback refs / `ref` as a prop (React 19) when you need the instance (focus, measure, scroll, third-party host node).
- **Effects:** `useEffect` / `useLayoutEffect` for post-commit side effects tied to that ref or external system — with **cleanup**. Not for deriving render output.
- **Lists:** `map` + stable `key`; do not query `.item` nodes to attach behavior.
- **Portals:** `createPortal` to render outside the parent DOM node — not manual `appendChild` of React output.
- **Forms:** controlled or uncontrolled React inputs (`value`/`onChange` or `defaultValue` + ref). Do not drive the form by hunting `form.elements` / querySelector unless integrating a non-React API.
- **Visibility / branches:** conditional JSX (`&&`, ternary), not `display` / `hidden` toggled via DOM APIs when React can unmount or flip props instead.

### Escape hatches (allowed when justified)

Use refs + effects (cleanup required) only when React has no good declarative API:

- Focus, selection, scroll, resize/measure (`getBoundingClientRect`, `ResizeObserver`)
- Media, canvas, WebSocket, geolocation, and similar browser APIs
- Third-party **non-React** libraries that require a mount node
- Integrating with non-React legacy widgets

Rules for escape hatches:

1. Obtain the node via **ref**, never via global selectors into React’s tree.
2. Create / update / tear down in an **effect**; return a cleanup that disposes listeners and library instances.
3. Do **not** let the external code fight React over the same children/attributes React also controls.
4. Prefer a thin host component (`ChartHost`, `MapHost`) so the escape hatch stays localized.

```tsx
const ChartHost: React.FC<ChartHostProps> = ({ data, ...otherProps }) => {
  const hostRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const el = hostRef.current
    if (!el) {
      return
    }

    const chart = createThirdPartyChart(el, data)
    return () => {
      chart.destroy()
    }
  }, [data])

  return <div ref={hostRef} {...otherProps} />
}
```

### Mental check

Before writing DOM API code inside a component/hook: “Does React already expose this via props, state, JSX events, refs, or portal?” If yes → use that. If no → ref + effect escape hatch, scoped and cleaned up.

## Event handlers

- Never write callback functions inline in the markup.
- Declare them in the component body as **named arrow functions** with relevant memoizing and deps when necessary.

```tsx
const MyComponent: React.FC<MyComponentProps> = ({
  prop1 = 'default',
  ...otherProps
}) => {
  const handleClick = () => {}

  return <div onClick={handleClick} {...otherProps} />
}
```

## className on the outer wrapper

- If the outermost wrapper gets a CSS class: destructure `className` from props and apply it on that element.
- If the project has a class-merge utility (`clsx`, `cn`, etc.), use it. Otherwise **do not** destructure `className` — let it pass through the spread.

```tsx
const MyComponent: React.FC<MyComponentProps> = ({
  className,
  ...otherProps
}) => <div className={clsx(styles.MyComponent, className)} {...otherProps} />
```

## Dynamic `style` and custom attributes

- Prefer controlling CSS via **custom HTML attributes** (`data-*`) and **`dynamicStyle`**.
- When the component manipulates `style`: destructure it from props, build `dynamicStyle` as `React.CSSProperties`, pass it to the element.
- **Never** put raw CSS properties (e.g. `color`, `padding`, `margin`, `transform`) in `dynamicStyle` or other dynamic inline styles — always set **CSS custom properties** (`--*`) and consume them in CSS with `var()`.
- Decide `useMemo` (or not) when inline style identity would cause excess re-renders.
- Place `...style` first or last deliberately (defaults vs consumer overwrite).

```tsx
const MyComponent: React.FC<MyComponentProps> = ({
  style,
  amount,
  full,
  ...otherProps
}) => {
  const dynamicStyle: React.CSSProperties = {
    ...style,
    ...(amount && !full && { '--vui-bleed-amount': `var(--space-${amount})` }),
    // or ...style at the end to allow consumer overwrite
  }

  // [data-prop] is then used in css to customize style
  return <div style={dynamicStyle} data-prop={prop1} {...otherProps} />
}
```

## `data-*` attribute values

- Custom HTML attributes (`data-*`) always receive the strings **`"true"`** or **`"false"`**.
- Do **not** toggle attribute presence with booleans (`<div {...(bool && { "data-prop": bool })} />`).

```tsx
// data-prop becomes [data-prop="true"] or [data-prop="false"].
<div style={dynamicStyle} data-prop={prop1} {...otherProps} />
```

Folder and file placement: see [`filesystem.md`](filesystem.md).

## Checklist

- [ ] `const` named arrow function
- [ ] `React.FC` with props generic
- [ ] React utility types via `React.*` — no named imports (`FC`, `CSSProperties`, `ComponentPropsWithRef`, …)
- [ ] `ComponentNameProps` (export only if callers need it; no satellite union/alias exports — use `Props['prop']`)
- [ ] One-shot unions inlined on the prop; file-local aliases stay unexported
- [ ] Custom props: TSDoc + `@defaultValue` matching assigned default
- [ ] Prop types reused via indexed access / `typeof` — no redeclared copies
- [ ] Extends `React.ComponentPropsWithRef` / `React.ComponentPropsWithoutRef` of the outer wrapper when spreading
- [ ] Destructure + residual spread; spread order intentional
- [ ] Defaults in param list when possible
- [ ] Markup: `&&` for null branch; flat ternary otherwise
- [ ] Prefer `??` where applicable
- [ ] CSS modules → `styles` import; plain CSS → side-effect import
- [ ] Use configured TypeScript path aliases where applicable; otherwise recommend configuring them
- [ ] Avoid deep imports; import through the relative `index` module when available
- [ ] Outer wrapper `className`: merge with project util, else leave on spread
- [ ] Prefer project tools over custom/extra scripting
- [ ] Performance considered (memo / Suspense / etc. when warranted)
- [ ] No inline callbacks in JSX — named handlers in body
- [ ] Stay inside React: no `querySelector` / `getElementById` / `addEventListener` / `classList` / `innerHTML` / `createElement` on React-owned UI — use props, state, JSX events, refs, portals
- [ ] Imperative DOM only as escape hatch: ref + effect + cleanup (third-party host, focus, measure, etc.)
- [ ] Prefer `data-*` + `dynamicStyle: React.CSSProperties` (+ memo when needed); `dynamicStyle` sets only `--*` custom props, never raw CSS properties
- [ ] `data-*` values are `"true"` / `"false"` strings, not booleans
