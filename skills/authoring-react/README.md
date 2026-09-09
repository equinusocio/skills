# authoring-react

Personal conventions for **React + TypeScript UI**. Auto-applies when creating, editing, refactoring, or reviewing components, hooks, JSX/TSX, or props.

**Version:** 1.4.0 · **Hub:** [`SKILL.md`](./SKILL.md)

```bash
npx skills add equinusocio/skills --skill authoring-react
```

← [All skills](../../README.md)

## Example output

```tsx
import React from 'react'
import { Stack, Text } from '@vira-ui/react'
import clsx from 'clsx'
import styles from './status-panel.module.css'

export type StatusPanelProps = React.ComponentPropsWithRef<typeof Stack> & {
  /** Emphasize the panel. @defaultValue false */
  accent?: boolean
}

export const StatusPanel: React.FC<StatusPanelProps> = ({
  accent = false,
  className,
  style,
  children,
  ...otherProps
}) => {
  const dynamicStyle: React.CSSProperties = {
    ...style,
    ...(accent && { '--status-panel-accent': 'var(--color-brand)' }),
  }

  return (
    <Stack
      className={clsx(styles.StatusPanel, className)}
      style={dynamicStyle}
      data-accent={accent ? 'true' : 'false'}
      gap="space-200"
      {...otherProps}
    >
      <Text className={styles.Title}>Status</Text>
      {children}
    </Stack>
  )
}
```

## Contents

| File | Role |
| --- | --- |
| [`SKILL.md`](./SKILL.md) | Hub: contract, router, out of scope |
| [`authoring.md`](./authoring.md) | Component shape, props, markup, handlers, `className` / `style` / `data-*` |
| [`filesystem.md`](./filesystem.md) | Component folders first; hooks/libs/utils same spirit — `.tsx` only when JSX |

## Highlights

- Named `const` arrow components typed with `React.FC`
- Utility types via `React.*` (`React.ComponentPropsWithRef`, `React.CSSProperties`, …) — no named imports from `'react'`
- Defaults in the parameter list; named handlers (no inline JSX callbacks)
- CSS modules → `styles` import; plain CSS → side-effect import
- Prefer `data-*` (`"true"` / `"false"` strings) + `dynamicStyle: React.CSSProperties`
- Folder: `/my-component` with `index.ts`, `my-component.tsx`, optional module CSS and subcomponents; hooks/libs use `.ts` when no JSX

## Out of scope

Non-UI TypeScript, Vue/Angular/Svelte, CSS conventions (use [`authoring-css`](../authoring-css/README.md)).
