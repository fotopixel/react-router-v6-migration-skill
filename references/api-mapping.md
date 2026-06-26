# React Router v5 → v6 API Mapping

## Hooks

| v5 | v6 | Import (during migration) | Notes |
|----|----|----|---|
| `useHistory()` | `useNavigate()` | `react-router-dom-v5-compat` | See navigation methods below |
| `useRouteMatch(path)` | `useMatch(path)` | `react-router-dom-v5-compat` | |
| `useParams<T>()` | `useParams()` | unchanged | Generic removed in v6 |
| `useLocation()` | `useLocation()` | unchanged | No API change |

## Navigation methods

| v5 (`history`) | v6 (`navigate`) |
|---|---|
| `history.push('/path')` | `navigate('/path')` |
| `history.push('/path', state)` | `navigate('/path', { state })` |
| `history.replace('/path')` | `navigate('/path', { replace: true })` |
| `history.goBack()` | `navigate(-1)` |
| `history.go(-2)` | `navigate(-2)` |
| `history.go(1)` | `navigate(1)` |

## Components

| v5 | v6 | Notes |
|----|----|----|
| `<Switch>` | `<Routes>` | |
| `<Route component={X} path="..." exact>` | `<Route element={<X />} path="...">` | `exact` removed |
| `<Route render={...}>` | `<Route element={<X />}>` | use hooks inside X |
| `<Redirect to="...">` | `<Navigate to="..." replace>` | add `replace` — v5 replaces by default |
| `<Redirect to="..." push>` | `<Navigate to="...">` | |
| `<NavLink exact>` | `<NavLink end>` | |
| `<NavLink activeClassName="x">` | `<NavLink className={({ isActive }) => isActive ? "x" : ""}>` | |
| `<NavLink activeStyle={...}>` | `<NavLink style={({ isActive }) => isActive ? ... : ...}>` | |

## Route props

| v5 prop | v6 equivalent |
|---|---|
| `exact` | removed (v6 is exact by default) |
| `strict` | removed |
| `component={X}` | `element={<X />}` |
| `render={({ match }) => <X id={match.params.id} />}` | `element={<X />}` + `useParams()` inside X |
| `children={fn}` | `element={<X />}` + hooks inside X |

## Wildcard / catch-all routes

```tsx
// v5
<Route component={NotFound} />  // no path = catch-all

// v6
<Route path="*" element={<NotFound />} />
```

## Programmatic redirect on render

```tsx
// v5
if (!isAuthorized) return <Redirect to="/login" />;

// v6
if (!isAuthorized) return <Navigate to="/login" replace />;
```

## withRouter (HOC — not used in this codebase)

```tsx
// v5
export default withRouter(MyComponent);

// v6 — use hooks directly inside the component instead
```

## StaticRouter (SSR)

```tsx
// v5
import { StaticRouter } from 'react-router-dom';

// v6
import { StaticRouter } from 'react-router-dom/server';
```
