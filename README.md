# skills

Personal [Agent Skills](https://skills.sh) for Cursor and compatible agents. Each skill auto-applies from the task — no `@`-mention required.

```bash
npx skills add equinusocio/skills
```

[![skills.sh](https://skills.sh/b/equinusocio/skills)](https://skills.sh/equinusocio/skills)

## Skills

| Skill | Docs | Triggers when… |
| --- | --- | --- |
| [`authoring-react`](./skills/authoring-react) | [README](./skills/authoring-react/README.md) | Creating, editing, refactoring, or reviewing React/TS UI (components, hooks, JSX/TSX, props) |
| [`authoring-css`](./skills/authoring-css) | [README](./skills/authoring-css/README.md) | Writing or changing stylesheets, CSS modules, nesting, selectors, colors, gradients, motion, `@property` |

Both can apply in the same task (e.g. new component + co-located CSS).

## Install

```bash
# all skills
npx skills add equinusocio/skills

# one skill
npx skills add equinusocio/skills --skill authoring-react
npx skills add equinusocio/skills --skill authoring-css

# list
npx skills add equinusocio/skills --list

# local checkout
npx skills add ./path/to/skills
```

Requires the [skills](https://skills.sh) CLI (`npx skills`).

## How to use

1. Install (above).
2. Work as usual — agent matches task to skill descriptions.
3. Optional: name a skill explicitly to force it.

### Shared hard contract

- Apply on **every** matching change — no opt-out for convenience.
- **Consumer project guidelines gate:** before writing, agent checks whether *your* project already documents guidelines / style guides / lint rules for that work.
  - None found → follow this skill.
  - Found → agent asks which source to follow (this skill, project guidelines, or a mix) and waits.
- **Force majeure:** skip or bend a rule only if you explicitly override for the task, or if following it would break the project’s established pattern / build.

## Repo layout

```
skills/
  <name>/SKILL.md      # hub (installed)
  <name>/README.md     # human docs (this catalog links here)
  <name>/*.md          # refs agent loads on demand
evals/
  <name>/              # trigger queries + eval prompts (not installed)
```

Add skills under `skills/<name>/` with matching `evals/<name>/`.

## License

MIT (see skill frontmatter). Author: [equinusocio](https://github.com/equinusocio).
