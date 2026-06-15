---
name: nextjs-app-router
description: Write correct, idiomatic Next.js 16 code using the App Router — Server vs Client components, data fetching, routing conventions, image/font optimization, and common pitfalls to avoid
---

# Next.js App Router Best Practices

We use Next.js 16 with the App Router. The App Router is fundamentally different from the
Pages Router — read `node_modules/next/dist/docs/` before writing any unfamiliar API.

## Server vs. Client Components

The most important mental model in App Router.

- **Default to Server Components.** Every file in `app/` is a Server Component unless
  it declares `'use client'` at the top. Server Components run only on the server: they
  can be `async`, access back-end resources directly, and produce zero JS bundle impact.
- **Add `'use client'` only when the component needs:**
  - React hooks (`useState`, `useEffect`, `useContext`, `useRef`, …)
  - Browser APIs (`window`, `document`, `localStorage`, …)
  - Event listeners (`onClick`, `onChange`, `onSubmit`, …)
  - Third-party libraries that depend on the above
- **AVOID** marking a parent `'use client'` to give a child interactivity — instead, extract
  the interactive part into its own small Client Component and keep the parent as a Server Component.
- **NEVER** use `useState`, `useEffect`, or any React hook in a Server Component — this
  causes a hard build error.
- **NEVER** pass non-serializable values (functions, class instances, `Date` objects) from
  a Server Component as props to a Client Component. Strings, numbers, plain objects, and
  arrays are fine.

## Data Fetching

- **PREFER** `async` Server Components for data fetching — no `useEffect`, no loading state,
  no client round-trips:
  ```tsx
  // ✅ Server Component
  export default async function CatalogPage() {
    const products = await getProducts(); // direct call, no useEffect
    return <ProductGrid products={products} />;
  }
  ```
- **AVOID** `useEffect` for data fetching in Client Components. If data must come from the
  server, fetch it in a parent Server Component and pass it down as props.
- Use `fetch()` with Next.js cache options for external APIs:
  ```ts
  fetch(url, { next: { revalidate: 3600 } }); // ISR — revalidate every hour
  fetch(url, { cache: 'no-store' });            // always fresh (SSR)
  fetch(url);                                   // cached until manually invalidated (SSG)
  ```
- For static local data (e.g. product catalog from `lib/data.ts`), just import and use
  directly in Server Components — no fetch needed.

## Routing Conventions

File-based routing under `app/`. Key reserved filenames:

| File | Purpose |
|------|---------|
| `page.tsx` | Route UI — makes the segment publicly accessible |
| `layout.tsx` | Shared wrapper — persists across child navigations |
| `loading.tsx` | Instant loading UI shown via Suspense while page streams |
| `error.tsx` | Error boundary for the segment — **must** be a Client Component |
| `not-found.tsx` | UI shown when `notFound()` is called |
| `template.tsx` | Like layout but re-mounts on every navigation |

- **PREFER** `<Link href="…">` from `next/link` for all internal navigation. Never use `<a>`.
- **NEVER** import from `next/router` — that is Pages Router only. Use `next/navigation`:
  ```ts
  import { useRouter, usePathname, useSearchParams, redirect, notFound } from 'next/navigation';
  ```
- `useRouter`, `usePathname`, and `useSearchParams` are **Client Component only** hooks.
- Call `redirect('/path')` or `notFound()` inside Server Components to redirect or 404.

## Layouts

- `app/layout.tsx` is the root layout — it wraps every page. It must render `{children}`.
- Layouts **do not re-render** on navigation between sibling routes — do not put
  per-page logic there. Put per-page logic in `page.tsx`.
- **AVOID** fetching page-specific data in a layout; layouts don't receive route params
  reliably. Fetch in the `page.tsx` instead.
- Nest layouts for route groups that share UI: `app/(catalog)/layout.tsx`.

## Images

- **ALWAYS** use `<Image>` from `next/image` instead of `<img>`. Never use raw `<img>` for
  content images — you lose automatic optimization, lazy-loading, and CLS prevention.
  ```tsx
  import Image from 'next/image';
  <Image src="/assets/brand/odara-hero.jpg" alt="Odara" width={800} height={600} />
  ```
- Use `fill` + a positioned parent for images whose intrinsic size is unknown:
  ```tsx
  <div className="relative h-64 w-full">
    <Image src={src} alt={alt} fill className="object-cover" />
  </div>
  ```
- Add `priority` to above-the-fold images (hero, first product row) to avoid LCP penalty.

## Fonts

- **ALWAYS** load fonts via `next/font/google` — never `<link>` tags or CSS `@import`.
  `next/font` self-hosts, eliminates layout shift, and removes the Google DNS request.
  ```ts
  // app/layout.tsx
  import { Cormorant_Garamond, Jost } from 'next/font/google';

  const cormorant = Cormorant_Garamond({
    subsets: ['latin'],
    weight: ['300', '400', '600'],
    variable: '--font-cormorant',
  });
  const jost = Jost({ subsets: ['latin'], variable: '--font-jost' });
  ```
- Attach font variables to `<html>` and reference them in CSS: `font-family: var(--font-jost)`.

## Metadata

- Export a `metadata` object from `page.tsx` or `layout.tsx` for static metadata:
  ```ts
  export const metadata: Metadata = {
    title: 'Odara — Arte em Presentear',
    description: 'Presentes artesanais feitos com amor.',
  };
  ```
- Use `generateMetadata()` (async function) when metadata depends on route params or
  fetched data.
- **NEVER** use `<title>` or `<meta>` tags directly inside JSX — Next.js manages the `<head>`.

## Streaming & Suspense

- Wrap slow Server Components in `<Suspense fallback={<Skeleton />}>` to stream the page
  progressively — fast content appears immediately, slow content fills in later.
- `loading.tsx` is a shorthand that wraps the whole `page.tsx` in a Suspense boundary
  automatically.

## Environment Variables

- Variables prefixed `NEXT_PUBLIC_` are bundled into the client JS and readable in the
  browser. All others are server-only.
- **NEVER** put secrets (API keys, tokens) in `NEXT_PUBLIC_` variables.
- Access env vars via `process.env.VAR_NAME` — no additional imports needed.

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `useRouter` / `usePathname` in a Server Component | Move to a `'use client'` component |
| Importing `next/router` | Change to `next/navigation` |
| `useEffect` to fetch data | Use an `async` Server Component instead |
| `<img>` for content images | Replace with `next/image` `<Image>` |
| Google Fonts via `<link>` | Use `next/font/google` |
| `<title>` / `<meta>` in JSX | Use `export const metadata` instead |
| Passing a function prop Server → Client | Extract as a Client Component or use a Server Action |
| Global interactivity in `app/layout.tsx` | Keep layout minimal; interactivity lives in Client Components |
