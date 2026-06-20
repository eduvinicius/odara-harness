---
name: react-spa
description: >
  Write correct, idiomatic React SPA code using Vite and React Router v7 — entry point
  setup, router configuration (createBrowserRouter), protected routes, Outlet-based
  layouts, navigation hooks, and environment variables. Use whenever the task involves
  routing, Vite config, or SPA architecture. Do NOT use for Next.js projects.
---

# React SPA with Vite + React Router v7

## Entry Point

`src/main.tsx` mounts the app. `QueryClientProvider` and `RouterProvider` live here
(or split between `main.tsx` and `App.tsx`).

```tsx
// src/main.tsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { QueryClientProvider } from '@tanstack/react-query'
import { queryClient } from './lib/queryClient'
import App from './App'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  </StrictMode>
)
```

---

## Router Setup

Use `createBrowserRouter` (data router API). **Never use `<BrowserRouter>`** — it is the
legacy API and conflicts with the data router.

```tsx
// src/App.tsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom'

const router = createBrowserRouter([
  { path: '/login', element: <LoginPage /> },
  {
    element: <ProtectedRoute />,        // session guard
    children: [
      {
        element: <AdminShell />,        // persistent layout (sidebar, topbar)
        children: [
          { path: '/',                  element: <DashboardPage /> },
          { path: '/products',          element: <ProductListPage /> },
          { path: '/products/new',      element: <ProductNewPage /> },
          { path: '/products/:id/edit', element: <ProductEditPage /> },
        ],
      },
    ],
  },
])

export default function App() {
  return <RouterProvider router={router} />
}
```

---

## Protected Routes

Check for an active session; redirect to `/login` if none is found.

```tsx
// src/router/ProtectedRoute.tsx
import { useEffect, useState } from 'react'
import { Outlet, Navigate } from 'react-router-dom'
import { getSession } from '../lib/auth'

export function ProtectedRoute() {
  const [status, setStatus] = useState<'loading' | 'auth' | 'unauth'>('loading')

  useEffect(() => {
    getSession().then(session => {
      setStatus(session ? 'auth' : 'unauth')
    })
  }, [])

  if (status === 'loading') return <FullPageSpinner />
  return status === 'auth' ? <Outlet /> : <Navigate to="/login" replace />
}
```

Rules:
- Every non-public route must be nested under `<ProtectedRoute>`.
- Always verify with `getSession()` — never assume a session exists.
- Do not store the session in `localStorage` manually; Supabase Auth handles persistence.

---

## Layout with Outlet

Persistent layouts (sidebar, top bar) use `<Outlet />` to render child routes.

```tsx
// src/components/layout/AdminShell.tsx
import { Outlet } from 'react-router-dom'

export function AdminShell() {
  return (
    <div className="flex h-screen">
      <Sidebar />
      <main className="flex-1 overflow-auto p-6">
        <Outlet />
      </main>
    </div>
  )
}
```

---

## Navigation Reference

| Need | API |
|---|---|
| Internal link in JSX | `<Link to="/products">` from `react-router-dom` |
| Navigate after form submit | `const navigate = useNavigate(); navigate('/products')` |
| Read URL param (e.g. `:id`) | `const { id } = useParams()` |
| Read current pathname | `const location = useLocation()` |
| Read/update query string | `const [params, setParams] = useSearchParams()` |
| Redirect in JSX | `<Navigate to="/login" replace />` |

Never use `<a href="...">` for internal routes — full page reload, breaks SPA.  
Never use `window.location.href` for navigation — resets React state.

---

## Environment Variables

Vite exposes env vars via `import.meta.env`. Only `VITE_`-prefixed vars are available
at runtime — they are bundled into the client JS (treat them as public).

```ts
// ✅ Correct
const url = import.meta.env.VITE_SUPABASE_URL

// ❌ Wrong — process.env is not available in Vite at runtime
const url = process.env.VITE_SUPABASE_URL
```

Add types in `src/vite-env.d.ts`:
```ts
interface ImportMetaEnv {
  readonly VITE_SUPABASE_URL: string
  readonly VITE_SUPABASE_ANON_KEY: string
}
interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

**Security**: `VITE_*` vars are public. Never put a service role key, secret API key,
or any private credential in a `VITE_` variable.

---

## Common Pitfalls

| Mistake | Correct approach |
|---|---|
| `<BrowserRouter>` alongside `createBrowserRouter` | Use only `<RouterProvider router={router}>` |
| `import { Link } from 'react-router'` | Import from `'react-router-dom'`, not `'react-router'` |
| `useNavigate()` outside `<RouterProvider>` | Must be inside a component rendered within the router |
| `process.env.VITE_*` | `import.meta.env.VITE_*` |
| `useParams()` result used without null check | Returns `string \| undefined` — guard before use |
| Fetching data in `useEffect` | Use `useQuery` from TanStack Query |
| Sharing filter state via React props | Use `useSearchParams()` — makes filters bookmarkable |
