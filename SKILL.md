---
name: react-router-v6-migration
description: >
  Migrate React Router v5 code to v6 in this codebase using the react-router-dom-v5-compat
  gradual migration strategy. Use this skill whenever you are asked to migrate a feature,
  component, page, or hook to React Router v6, update router imports, replace useHistory
  with useNavigate, convert Switch/Route to Routes, or work with react-router-dom-v5-compat.
  Also trigger when the user mentions "react-router migration", "v5 to v6", "upgrade router",
  or asks to "use v6 APIs" in a component.
---

# React Router v5 → v6 Migration Skill

## Project Context

This codebase is in the middle of a **gradual migration** from React Router v5 to v6.

- **Infrastructure already in place**: `CompatRouter` wraps the app in `Main/index.tsx`, allowing v5 and v6 APIs to run in parallel.
- **Compat package**: `react-router-dom-v5-compat@6.x` — during migration, import v6 APIs from here, not from `react-router-dom` directly.
- **After full migration**: remove the compat package and import from `react-router-dom` directly.

## Migration Strategy

Migrate **bottom-up**: start with leaf components (pages, hooks), work toward route definitions (Switch → Routes). This lets you ship each component independently without breaking siblings.

### Per-feature checklist

- [ ] Update navigation hooks (`useHistory` → `useNavigate`)
- [ ] Update imports to come from `react-router-dom-v5-compat`
- [ ] Update `<Link>` and `<NavLink>` if needed
- [ ] Update route definitions (`<Switch>` → `<Routes>`, `component` prop → `element` prop)
- [ ] Replace `<Redirect>` with `<Navigate>`
- [ ] Remove `exact` props (v6 is exact by default)

---

## API Transformations

### 1. Navigation: `useHistory` → `useNavigate`

This is the most common change — 66+ instances in the codebase.

```tsx
// BEFORE (v5)
import { useHistory } from 'react-router';

const history = useHistory();
history.push('/some/path');
history.push('/path', state);
history.replace('/some/path');
history.goBack();
history.go(-2);

// AFTER (v6) — import from compat during migration
import { useNavigate } from 'react-router-dom-v5-compat';

const navigate = useNavigate();
navigate('/some/path');
navigate('/path', { state });
navigate('/some/path', { replace: true });
navigate(-1);
navigate(-2);
```

### 2. Route definitions: `<Switch>` → `<Routes>`

Only do this once ALL descendant components have been migrated to v6 APIs.

```tsx
// BEFORE (v5)
import { Redirect, Route, Switch } from 'react-router';

<Switch>
  <Route component={Home} exact path={routes.home} />
  <Route component={Settings} exact path={routes.settings} />
  <Redirect to={routes.home} />
</Switch>

// AFTER (v6)
import { Navigate, Route, Routes } from 'react-router-dom-v5-compat';

<Routes>
  <Route element={<Home />} path={routes.home} />
  <Route element={<Settings />} path={routes.settings} />
  <Route path="*" element={<Navigate to={routes.home} replace />} />
</Routes>
```

Key differences:
- `component={X}` → `element={<X />}`
- `exact` removed (v6 routes are exact by default unless they end with `/*`)
- `<Redirect>` → `<Navigate>` (use `replace` prop to match v5's default replace behavior)
- Catch-all redirect goes in a `path="*"` route

### 3. Conditional routes (with feature flags)

The codebase heavily uses conditional routes. This pattern migrates cleanly:

```tsx
// BEFORE
<Switch>
  {canAccessFeature ? (
    <Route component={FeaturePage} exact path={routes.feature} />
  ) : null}
  <Route component={DefaultPage} exact path={routes.default} />
  <Redirect to={defaultRoute} />
</Switch>

// AFTER
<Routes>
  {canAccessFeature ? (
    <Route element={<FeaturePage />} path={routes.feature} />
  ) : null}
  <Route element={<DefaultPage />} path={routes.default} />
  <Route path="*" element={<Navigate to={defaultRoute} replace />} />
</Routes>
```

### 4. `<Redirect>` → `<Navigate>`

```tsx
// BEFORE
import { Redirect } from 'react-router';
<Redirect to="/some/path" />
<Redirect to="/some/path" push />  // push (not replace)

// AFTER
import { Navigate } from 'react-router-dom-v5-compat';
<Navigate to="/some/path" replace />  // v5 Redirect defaults to replace
<Navigate to="/some/path" />          // v6 Navigate defaults to push
```

> ⚠️ The default behavior differs: `<Redirect>` replaces by default; `<Navigate>` pushes by default. Always add `replace` when converting a `<Redirect>`.

### 5. Route params: `useParams`

No change needed — `useParams` works the same in both versions. Just update the import:

```tsx
// BEFORE
import { useParams } from 'react-router';

// AFTER
import { useParams } from 'react-router-dom-v5-compat';
// or keep from 'react-router-dom' — both work with CompatRouter
```

### 6. `useLocation`

No API change. Update the import source:

```tsx
// BEFORE
import { useLocation } from 'react-router';

// AFTER
import { useLocation } from 'react-router-dom-v5-compat';
```

### 7. `useRouteMatch` → `useMatch`

```tsx
// BEFORE
import { useRouteMatch } from 'react-router';
const match = useRouteMatch('/users/:id');

// AFTER
import { useMatch } from 'react-router-dom-v5-compat';
const match = useMatch('/users/:id');
// Note: useMatch returns null (no match) or a match object — same shape as v5
```

### 8. `NavLink` with `exact`

The codebase uses `exact` on `NavLink` (e.g. `AnimatedLinkContainer.tsx`):

```tsx
// BEFORE
<NavLink exact to={path}>...</NavLink>

// AFTER
<NavLink end to={path}>...</NavLink>
// 'exact' → 'end'
```

If using `activeClassName` or `activeStyle`:

```tsx
// BEFORE
<NavLink activeClassName="is-active" to={path}>...</NavLink>

// AFTER
<NavLink className={({ isActive }) => isActive ? 'is-active' : ''} to={path}>...</NavLink>
```

### 9. Imports during migration

While migrating, use `react-router-dom-v5-compat` for all v6 APIs. **Do not** mix-import v6 hooks from `react-router-dom` before migration is complete — the compat package ensures v5 and v6 state stays synchronized.

```tsx
// During migration — v6 APIs come from compat package
import { useNavigate, useMatch, Navigate, Routes, Route } from 'react-router-dom-v5-compat';

// v5 APIs stay from react-router / react-router-dom (untouched siblings)
import { useHistory, Switch, Route } from 'react-router';
```

---

## Parent routes with nested routes

When a feature has child routes rendered by a sub-component, the parent route needs a `/*` suffix to allow child matching:

```tsx
// Parent (in SecuredRoutes or top-level routing)
// BEFORE
<Route path={routes.accounting} component={Accounting} />

// AFTER — add /* to allow nested routes inside Accounting
<Route path={`${routes.accounting}/*`} element={<Accounting />} />
```

Inside the feature component, child routes use **relative paths**:

```tsx
// Accounting/index.tsx
// BEFORE
<Switch>
  <Route component={ExportsHome} exact path={routes.exports} />       // absolute
  <Route component={CreateExport} exact path={routes.newExport} />   // absolute
</Switch>

// AFTER — paths relative to the parent route match
<Routes>
  <Route element={<ExportsHome />} path="exports" />
  <Route element={<CreateExport />} path="exports/new" />
</Routes>
```

Note: The existing `routes.ts` constants (absolute paths) still work — v6 Routes accept both absolute and relative paths. Only switch to relative paths if that's the explicit goal of the migration.

---

## `history.push` with state

```tsx
// BEFORE
history.push('/path', { from: 'dashboard' });

// AFTER
navigate('/path', { state: { from: 'dashboard' } });
```

Accessing state (no change):
```tsx
const location = useLocation();
const state = location.state as { from: string };
```

---

## What NOT to change

- **`routes.ts` files** — the string path constants are just strings; no changes needed.
- **`useParams` call sites** — API is identical; just update the import.
- **`useLocation` call sites** — API is identical; just update the import.
- **The `CompatRouter` in `Main/index.tsx`** — leave this in place until the full app is migrated.

---

## Reference

See `references/api-mapping.md` for a compact lookup table of every v5 → v6 change.
