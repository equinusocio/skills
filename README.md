# react-authoring-skills

Agent Skills for Cursor (and compatible agents) that encode personal conventions when authoring **React + TypeScript UI** and **CSS**.

Once installed, skills **auto-apply from the task** — no need to `@`-mention or load them by hand. The agent matches the request (e.g. “add a Button component”, “fix panel.module.css”) and follows the matching skill.

Distribute with:

```bash
npx skills add equinusocio/react-authoring-skills
```

## Skills at a glance

| Skill | Path | Triggers when… |
| --- | --- | --- |
| [`authoring-react`](./skills/authoring-react) | `skills/authoring-react` | Creating, editing, refactoring, or reviewing React/TS UI (components, hooks, JSX/TSX, props) |
| [`authoring-css`](./skills/authoring-css) | `skills/authoring-css` | Writing or changing stylesheets, CSS modules, nesting, selectors, colors, gradients, motion, `@property` |

Both can apply in the same task (e.g. new component + co-located CSS).

## Install

### All skills in this repo

```bash
npx skills add equinusocio/react-authoring-skills
```

### One skill

```bash
npx skills add equinusocio/react-authoring-skills --skill authoring-react
npx skills add equinusocio/react-authoring-skills --skill authoring-css
```

### List available skills

```bash
npx skills add equinusocio/react-authoring-skills --list
```

### Local checkout (development)

```bash
npx skills add ./path/to/react-authoring-skills
```

Requires the [skills](https://skills.sh) CLI (`npx skills`). Skills land in the agent’s skill path for the current user or project, depending on how you install.

## How to use

1. Install (above).
2. Ask the agent to write or change React/TS UI and/or CSS as usual.
3. The agent should load the skill automatically from the description triggers — **no** `/skill`, `@skill`, or “use authoring-react” required.
4. Optional: name the skill explicitly if you want to force it (`authoring-react`, `authoring-css`).

### Hard contract (both skills)

- Apply on **every** matching change — no opt-out for convenience.
- **Consumer project guidelines gate:** before writing, the agent checks whether *your* project already documents guidelines / style guides / lint rules for that work.
  - None found → follow this skill.
  - Found → agent asks which source to follow (this skill, project guidelines, or a mix) and waits.
- **Force majeure:** skip or bend a rule only if you explicitly override for the task, or if following it would break the project’s established pattern / build.

### Out of scope (reminders)

| Skill | Does not cover |
| --- | --- |
| `authoring-react` | Non-UI TypeScript, Vue/Angular/Svelte, CSS conventions (use `authoring-css`) |
| `authoring-css` | React component structure (use `authoring-react`), SCSS/Less-only dialects, Tailwind class strings alone |

## What they contain

Each skill is a thin **hub** (`SKILL.md`) plus **refs** the agent reads on demand.

### `authoring-react` (v1.0.0)

| File | Role |
| --- | --- |
| [`SKILL.md`](./skills/authoring-react/SKILL.md) | Hub: contract, router, out of scope |
| [`authoring.md`](./skills/authoring-react/authoring.md) | Component shape, props, markup, handlers, `className` / `style` / `data-*` |
| [`filesystem.md`](./skills/authoring-react/filesystem.md) | Kebab-case folders, `index.ts`, co-located CSS and subcomponents |

**Highlights:**

- Named `const` arrow components typed with `React.FC` / `FC`
- `ComponentNameProps` extending `ComponentPropsWithRef` / `WithoutRef` of the outer wrapper when spreading
- Defaults in the parameter list; named handlers (no inline JSX callbacks)
- CSS modules → `styles` import; plain CSS → side-effect import
- Prefer `data-*` (`"true"` / `"false"` strings) + `dynamicStyle: React.CSSProperties`
- Folder: `/my-component` with `index.ts`, `my-component.tsx`, optional module CSS and subcomponents

### `authoring-css` (v1.3.0)

| File | Role |
| --- | --- |
| [`SKILL.md`](./skills/authoring-css/SKILL.md) | Hub: contract, router, out of scope |
| [`authoring.md`](./skills/authoring-css/authoring.md) | Classes, nesting, Baseline, colors, shorthand/longhand, motion, `@property` |

**Highlights:**

- PascalCase classes (`.MyClass`); root class matches component name (`.Stack` for `Stack`)
- Module children: element name only (`.Content`) as top-level siblings — no `Component_` prefix; do not nest class-in-class for DOM structure
- Native CSS nesting only for that selector’s own variants/states/media/`&` rules; modern selectors (`:has`, `:is`, `:where`, …) when useful
- Prefer Baseline / project browserslist; ask when target unclear
- Hardcoded colors: OKLCH/OKLAB (esp. gradients); derive/alpha with relative colors — no `color-mix()`
- Prefer shorthand (≤5 values); longhand only when shorthand would need >5 values; no autoprefixer-redundant prefixes
- Motion on performant props; `@property` in `*.props.css` imported from the component stylesheet

## Repo layout

```
skills/
  authoring-react/SKILL.md
  authoring-react/authoring.md
  authoring-react/filesystem.md
  authoring-css/SKILL.md
  authoring-css/authoring.md
evals/
  authoring-react/
  authoring-css/
```

- **`skills/<name>/`** — publishable Agent Skills (hub + refs).
- **`evals/<name>/`** — trigger queries and eval prompts for skill quality (not installed as skills).

Add more skills under `skills/<name>/` with matching `evals/<name>/` when contributing.

## License

MIT (see skill frontmatter). Author: [equinusocio](https://github.com/equinusocio).
