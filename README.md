# React Router v5 → v6 Migration Skill

An agent skill for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and [Cursor](https://cursor.com) that guides incremental migration from React Router v5 to v6 using the official [`react-router-dom-v5-compat`](https://www.npmjs.com/package/react-router-dom-v5-compat) package.

The skill teaches the agent a bottom-up strategy: migrate leaf components first, route definitions last, and run v5 and v6 APIs side by side until the app is fully upgraded.

## What's in this repo

```
react-router-v6-migration-skill/
└── SKILL.md    # Skill instructions — strategy, API mappings, examples, edge cases
```

[`SKILL.md`](./SKILL.md) is loaded by the agent when migration work comes up. It covers:

- **Migration strategy** — bottom-up: pages and hooks first, `<Switch>` → `<Routes>` last
- **API transformations** — `useHistory` → `useNavigate`, `<Redirect>` → `<Navigate>`, `component` → `element`, and more
- **Compat package rules** — import v6 APIs from `react-router-dom-v5-compat` during migration, not directly from `react-router-dom`
- **Edge cases** — conditional routes, nested routes with `/*`, `NavLink` `exact` → `end`, navigation state, and what *not* to change

## Installation

Clone this repo into the skills directory for your tool. The folder name should match the skill (`react-router-v6-migration`).

### Claude Code

**Personal** (available in all projects):

```bash
git clone https://github.com/fotopixel/react-router-v6-migration-skill.git \
  ~/.claude/skills/react-router-v6-migration
```

**Project** (shared with the team via the repo):

```bash
git clone https://github.com/fotopixel/react-router-v6-migration-skill.git \
  .claude/skills/react-router-v6-migration
```

### Cursor

**Personal** (available in all projects):

```bash
git clone https://github.com/fotopixel/react-router-v6-migration-skill.git \
  ~/.cursor/skills/react-router-v6-migration
```

**Project** (shared with the team via the repo):

```bash
git clone https://github.com/fotopixel/react-router-v6-migration-skill.git \
  .cursor/skills/react-router-v6-migration
```

You can also add the repo as a git submodule in any of the paths above.

## Usage

Once installed, the agent picks up the skill when you ask to migrate routing code — for example:

- "Migrate this component to React Router v6"
- "Replace `useHistory` with `useNavigate`"
- "Convert this `<Switch>` to `<Routes>`"
- "React Router v5 to v6 migration"

You can also invoke it explicitly:

```
Use the react-router-v6-migration skill to migrate src/pages/Settings.tsx
```

## Prerequisites in your codebase

The skill assumes your app already uses the compat layer:

1. **`CompatRouter`** wraps the app (typically at the root layout)
2. **`react-router-dom-v5-compat@6.x`** is installed
3. v5 and v6 APIs can coexist until migration is complete

After the full app is migrated, remove the compat package and import directly from `react-router-dom`.

## Migration checklist

For each feature or component:

- [ ] Replace `useHistory` with `useNavigate`
- [ ] Update imports to `react-router-dom-v5-compat`
- [ ] Update `<Link>` / `<NavLink>` as needed (`exact` → `end`, `activeClassName` → render prop)
- [ ] Convert route definitions: `<Switch>` → `<Routes>`, `component` → `element`
- [ ] Replace `<Redirect>` with `<Navigate>` (add `replace` — defaults differ between v5 and v6)
- [ ] Remove `exact` props (v6 routes are exact by default)

Migrate route definitions only after descendant components already use v6 APIs.

## Key API changes

| v5 | v6 |
|----|-----|
| `useHistory()` | `useNavigate()` |
| `history.push(path)` | `navigate(path)` |
| `history.push(path, state)` | `navigate(path, { state })` |
| `history.replace(path)` | `navigate(path, { replace: true })` |
| `history.goBack()` | `navigate(-1)` |
| `<Switch>` + `<Route component={X}>` | `<Routes>` + `<Route element={<X />}>` |
| `<Redirect to="..." />` | `<Navigate to="..." replace />` |
| `useRouteMatch()` | `useMatch()` |
| `<NavLink exact>` | `<NavLink end>` |

See [`SKILL.md`](./SKILL.md) for full before/after examples, nested routing patterns, and things to leave unchanged.

## License

See the repository for license details.
