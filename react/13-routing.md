# React: Routing

React has no built-in router. Main options:

- React Router (v8, June 2026): the most used; works as a plain library or as a full framework (the successor of Remix).
- TanStack Router: fully type-safe routes and search params, file-based or code-based; TanStack Start is its full-stack framework.
- Next.js App Router: file-system routing with RSC (see [Ecosystem & Tooling](18-ecosystem-tooling.md)).

## React Router modes

| Mode | Setup | What you get |
| --- | --- | --- |
| Declarative | `<BrowserRouter>` + `<Routes>` | URL matching, links, params; nothing else |
| Data | `createBrowserRouter` + `<RouterProvider>` | + loaders, actions, pending UI, fetchers, errors |
| Framework | Vite plugin + `app/routes.ts` | + SSR/SPA/prerender, code splitting, generated route types, middleware, RSC |

v8 is ESM-only and needs React 19.2.7+, Node 22.22+, and Vite 7+. Everything is imported from `react-router` (`react-router-dom` is gone).

## Declarative mode

```tsx
import { BrowserRouter, Routes, Route, Link, NavLink, Outlet, Navigate } from 'react-router';

createRoot(root).render(
  <BrowserRouter>
    <Routes>
      <Route element={<Layout />}>                       {/* pathless layout route */}
        <Route index element={<Home />} />               {/* "/" */}
        <Route path="about" element={<About />} />
        <Route path="users/:userId" element={<User />} />
        <Route path="files/*" element={<Files />} />      {/* splat: params['*'] */}
        <Route path="docs/:lang?" element={<Docs />} />   {/* optional segment */}
        <Route path="old" element={<Navigate to="/about" replace />} />
        <Route path="*" element={<NotFound />} />
      </Route>
    </Routes>
  </BrowserRouter>,
);

function Layout() {
  return (
    <>
      <nav>
        <NavLink to="/" end className={({ isActive }) => (isActive ? 'active' : '')}>Home</NavLink>
        <Link to="/about">About</Link>
      </nav>
      <Outlet />                                         {/* child route renders here */}
    </>
  );
}
```

## Hooks

```tsx
import { useParams, useNavigate, useSearchParams, useLocation } from 'react-router';

function User() {
  const { userId } = useParams();                       // string | undefined
  const navigate = useNavigate();
  const [searchParams, setSearchParams] = useSearchParams();
  const location = useLocation();                       // { pathname, search, hash, state, key }

  const tab = searchParams.get('tab') ?? 'profile';
  setSearchParams({ tab: 'posts' });                    // ?tab=posts (adds a history entry)
  setSearchParams((p) => { p.set('page', '2'); return p; }, { replace: true });

  navigate('/users');                                   // push
  navigate(-1);                                         // back
  navigate('/login', { replace: true, state: { from: location.pathname } });
}
```

Keep filter, sort, page, and tab state in search params: it survives a reload, can be shared as a link, and the back button works.

## Data mode

```tsx
import { createBrowserRouter, RouterProvider, redirect, useLoaderData, Form, useNavigation } from 'react-router';

const router = createBrowserRouter([
  {
    path: '/',
    Component: Root,
    ErrorBoundary: RootError,                            // catches loader/action/render errors
    children: [
      {
        path: 'contacts/:id',
        loader: async ({ params, request }) => {
          const res = await fetch(`/api/contacts/${params.id}`, { signal: request.signal });
          if (res.status === 404) throw new Response('Not found', { status: 404 });
          return res.json();                             // → useLoaderData()
        },
        action: async ({ request, params }) => {         // POST/PUT/DELETE from <Form>
          const form = await request.formData();
          await updateContact(params.id!, Object.fromEntries(form));
          return redirect(`/contacts/${params.id}`);     // loaders revalidate automatically
        },
        Component: Contact,
      },
      {
        path: 'contacts/:id/edit',
        lazy: { Component: async () => (await import('./ContactEditor')).default }, // code-split route
      },
    ],
  },
]);

createRoot(root).render(<RouterProvider router={router} />);

function Contact() {
  const contact = useLoaderData() as ContactType;
  const navigation = useNavigation();                   // 'idle' | 'loading' | 'submitting'
  return (
    <Form method="post">                                {/* submits to the route action, no fetch code */}
      <input name="name" defaultValue={contact.name} />
      <button disabled={navigation.state === 'submitting'}>Save</button>
    </Form>
  );
}
```

```tsx
// fetchers: call loaders/actions without navigating (like buttons, inline edits)
const fetcher = useFetcher();
<fetcher.Form method="post" action="/favorite">
  <button name="favorite" value="true">{fetcher.state !== 'idle' ? '…' : '★'}</button>
</fetcher.Form>
```

Loaders for nested routes run in parallel, before rendering, which removes the fetch-in-effect waterfalls.

## Framework mode

```console
$ npx create-react-router@latest my-app
```

```ts
// app/routes.ts
import { type RouteConfig, index, route, layout, prefix } from '@react-router/dev/routes';

export default [
  index('routes/home.tsx'),
  route('about', 'routes/about.tsx'),
  layout('routes/dashboard-layout.tsx', [
    route('dashboard', 'routes/dashboard.tsx'),
    ...prefix('projects', [
      index('routes/projects.tsx'),
      route(':pid', 'routes/project.tsx'),
    ]),
  ]),
] satisfies RouteConfig;
```

```tsx
// app/routes/project.tsx: a route module
import type { Route } from './+types/project';      // generated types per route
import { data } from 'react-router';
import { userContext } from '~/context';

export const middleware: Route.MiddlewareFunction[] = [authMiddleware]; // stable in v8

export async function loader({ params, context }: Route.LoaderArgs) {   // runs on the server
  const user = context.get(userContext);
  const project = await db.projects.find(params.pid, user.id);
  if (!project) throw data('Not found', { status: 404 });
  return { project };
}

export async function clientLoader({ serverLoader }: Route.ClientLoaderArgs) {
  return { ...(await serverLoader()), cachedAt: Date.now() };          // optional browser-side loader
}

export async function action({ request }: Route.ActionArgs) { /* … */ }

export function meta({ data }: Route.MetaArgs) {
  return [{ title: data?.project.name ?? 'Project' }];
}

export default function Project({ loaderData }: Route.ComponentProps) {
  return <h1>{loaderData.project.name}</h1>;                         // fully typed
}

export function ErrorBoundary({ error }: Route.ErrorBoundaryProps) { /* … */ }
```

```ts
// react-router.config.ts
import type { Config } from '@react-router/dev/config';
export default {
  ssr: true,                                   // false → SPA mode (only client loaders)
  prerender: ['/', '/about'],                  // static HTML at build time
} satisfies Config;
```

```ts
// middleware: authentication, logging, shared context
import { createContext, redirect } from 'react-router';
export const userContext = createContext<User>();

export const authMiddleware: Route.MiddlewareFunction = async ({ request, context }, next) => {
  const user = await getUserFromSession(request);
  if (!user) throw redirect('/login');
  context.set(userContext, user);
  return next();                                // runs child middleware + loaders; you can wrap the response
};
```

Type-safe links: `href('/projects/:pid', { pid })` builds URLs from the route config and fails type-checking on typos.

## Protected routes (client side)

```tsx
function RequireAuth({ children }: { children: React.ReactNode }) {
  const { user } = useAuth();
  const location = useLocation();
  if (!user) return <Navigate to="/login" replace state={{ from: location }} />;
  return children;
}
```

Hiding UI on the client is UX, not security. Enforce access on the server, in middleware or loaders, and in the API.

<!-- nav -->
---

← [React: Styling](12-styling.md) · [Index](README.md) · [React: Data Fetching & State Libraries](14-data-state-libs.md) →
<!-- nav -->
