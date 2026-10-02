# React: Ecosystem & Tooling

## Version timeline

| Version | Year | Headline features |
| --- | --- | --- |
| 16.8 | 2019 | Hooks |
| 17 | 2020 | new JSX transform, events attached to the root instead of `document` |
| 18 | 2022 | concurrent rendering, automatic batching, `useTransition`, `useId`, `useSyncExternalStore`, streaming SSR |
| 19.0 | 2024 | Actions, `useActionState`, `useOptimistic`, `useFormStatus`, `use`, ref as a prop, `<Context>` provider, metadata/stylesheet hoisting, RSC stable |
| 19.1 | 2025 | owner stacks (`captureOwnerStack`), Suspense fixes |
| 19.2 | 2025 | `<Activity>`, `useEffectEvent`, `cacheSignal`, Performance Tracks, Partial Pre-rendering |
| Compiler 1.0 | 2025 | automatic memoization, compiler lint rules |
| 19.3 | 2026 | `<ViewTransition>` + `addTransitionType`, Fragment refs, `browser()`, Trusted Types, `<Context>` in RSC |

React is governed by the React Foundation (Linux Foundation) since February 2026.

## Vite

The standard build tool for client-side React. Vite 8 uses Rolldown (a Rust bundler) for dev and build.

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'node:path';

export default defineConfig({
  plugins: [react()],
  resolve: { alias: { '@': path.resolve(import.meta.dirname, 'src') } }, // also set tsconfig "paths"
  server: {
    port: 3000,
    proxy: { '/api': { target: 'http://localhost:8080', changeOrigin: true } }, // avoid CORS in dev
  },
  build: { sourcemap: true },
});
```

```bash
# .env, .env.local, .env.production: only VITE_* variables reach the client
VITE_API_URL=https://api.example.com
DB_PASSWORD=secret              # NOT exposed to client code (and must never be)
```

```ts
const url = import.meta.env.VITE_API_URL;
if (import.meta.env.DEV) { /* dev only, removed from prod builds */ }
// types: interface ImportMetaEnv { readonly VITE_API_URL: string } in src/vite-env.d.ts
```

## Linting and formatting

```js
// eslint.config.js (flat config)
import js from '@eslint/js';
import tseslint from 'typescript-eslint';
import reactHooks from 'eslint-plugin-react-hooks';
import reactRefresh from 'eslint-plugin-react-refresh';
import jsxA11y from 'eslint-plugin-jsx-a11y';
import { defineConfig } from 'eslint/config';

export default defineConfig([
  js.configs.recommended,
  tseslint.configs.recommended,
  reactHooks.configs.flat.recommended,      // rules of hooks + exhaustive-deps + React Compiler rules
  reactRefresh.configs.vite,                // warns when a file breaks Fast Refresh
  jsxA11y.flatConfigs.recommended,
]);
```

Format with Prettier. Alternatives that are much faster: Biome (lint + format in one Rust tool) and Oxlint. Next.js 16 removed `next lint`; run ESLint or Biome directly.

## Next.js (App Router)

The most used full-stack React framework: RSC, Server Actions, file-based routing, streaming, and image/font optimization. Next.js 16 makes Turbopack the default bundler and requires Node 20.9+.

### File conventions

```text
app/
├── layout.tsx            # root layout (required): <html><body>{children}</body></html>
├── page.tsx              # "/"
├── loading.tsx           # Suspense fallback for the segment
├── error.tsx             # error boundary ('use client'), gets { error, reset }
├── not-found.tsx         # notFound() target
├── global-error.tsx      # errors in the root layout
├── (marketing)/          # route group: organizes files, not part of the URL
│   └── pricing/page.tsx  # "/pricing"
├── blog/
│   ├── [slug]/page.tsx   # dynamic "/blog/:slug"
│   └── [...all]/page.tsx # catch-all
├── @modal/               # parallel route slot (needs default.tsx in v16)
├── api/users/route.ts    # route handler: export GET/POST(Request) → Response
proxy.ts                  # request interception (formerly middleware.ts)
next.config.ts
```

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next';
import Link from 'next/link';
import Image from 'next/image';
import { notFound } from 'next/navigation';

type Props = { params: Promise<{ slug: string }>; searchParams: Promise<Record<string, string>> };

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;                   // v16: params/searchParams are async (no sync access)
  return { title: (await getPost(slug))?.title };
}

export async function generateStaticParams() {      // prerender these slugs at build time
  return (await getSlugs()).map((slug) => ({ slug }));
}

export default async function Page({ params }: Props) {
  const { slug } = await params;
  const post = await getPost(slug);
  if (!post) notFound();
  return (
    <article>
      <Image src={post.cover} alt="" width={1200} height={630} priority />
      <Link href="/blog">← Back</Link>
    </article>
  );
}
```

```ts
// cookies/headers are async too
import { cookies, headers } from 'next/headers';
const token = (await cookies()).get('session')?.value;
```

### Cache Components (Next.js 16)

Caching is opt-in. Without `'use cache'`, code runs per request. Enable it with `cacheComponents: true`: static parts get prerendered into a shell (PPR), and dynamic parts stream in under `<Suspense>`.

```ts
// next.config.ts
const nextConfig = { cacheComponents: true, reactCompiler: true };
export default nextConfig;
```

```tsx
import { cacheLife, cacheTag } from 'next/cache';

export async function getProducts() {
  'use cache';                       // cache this function's result; the args form the cache key
  cacheLife('hours');                // built-in profiles: 'seconds' | 'minutes' | 'hours' | 'days' | 'weeks' | 'max'
  cacheTag('products');
  return db.products.findMany();
}
```

```ts
'use server';
import { revalidateTag, updateTag, refresh } from 'next/cache';

export async function editProduct(id: string, data: FormData) {
  await db.products.update(id, data);
  updateTag('products');               // Server Actions only: expire + read fresh data in the same request
  // revalidateTag('products', 'max'); // stale-while-revalidate; v16 requires a cacheLife profile arg
  // refresh();                        // re-render uncached data only (Server Actions only)
}
```

### proxy.ts

```ts
// proxy.ts: runs before routes (Node runtime). Good for redirects, rewrites, and cheap auth gates.
import { NextResponse, type NextRequest } from 'next/server';

export default function proxy(request: NextRequest) {
  if (!request.cookies.has('session')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
}
export const config = { matcher: ['/dashboard/:path*'] };
// still check auth in pages and actions; proxy is an optimization, not the only line of defense
```

## Other frameworks and targets

- React Router (framework mode): loaders and actions, SSR/SPA/prerender, RSC support. Runs on any Node or edge host (see [Routing](13-routing.md)).
- TanStack Start: full-stack framework built on TanStack Router, with type-safe server functions.
- Astro: content-first sites; React components hydrate as "islands" (`client:load`, `client:visible`).
- Waku: minimal RSC framework.
- Expo / React Native: native iOS/Android apps with React. Uses `<View>`, `<Text>`, and `<Pressable>` instead of DOM elements. Expo Router gives file-based navigation, and the React Compiler is on by default since SDK 54.
- Electron / Tauri: desktop apps with a React UI.

## Useful libraries by category

| Need | Libraries |
| --- | --- |
| Server data | TanStack Query, SWR, RTK Query, Apollo, Relay, tRPC |
| Client state | Zustand, Redux Toolkit, Jotai, XState |
| Forms | React Hook Form, TanStack Form, Conform (Server Actions) |
| Validation | Zod, Valibot, ArkType |
| Routing | React Router, TanStack Router, Next.js |
| UI primitives | Radix UI, React Aria, Base UI, shadcn/ui |
| Tables / lists | TanStack Table, TanStack Virtual, AG Grid |
| Charts | Recharts, visx, Nivo, ECharts |
| Animation | Motion, React Spring, `<ViewTransition>` |
| Dates | date-fns, Day.js, Temporal (native, rolling out in browsers) |
| i18n | react-i18next, FormatJS (react-intl), Lingui, next-intl |
| Auth | Auth.js, Better Auth, Clerk, Supabase Auth |
| Drag and drop | dnd kit, Pragmatic drag and drop |
| Rich text | Tiptap, Lexical, Slate |
| Errors / monitoring | Sentry, `react-error-boundary` |
| Docs / component workshop | Storybook, Ladle |

<!-- nav -->
---

← [React: Patterns, Accessibility & Security](17-patterns.md) · [Index](README.md) · [Home](../README.md) →
<!-- nav -->
