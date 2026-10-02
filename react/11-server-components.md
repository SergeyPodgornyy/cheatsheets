# React: Server Components & Server Functions

## Rendering strategies

| Strategy | Where HTML is built | When | Notes |
| --- | --- | --- | --- |
| CSR (SPA) | browser | at runtime | empty HTML + JS bundle; Vite default |
| SSR | server | per request | HTML first, then **hydration** makes it interactive |
| SSG / prerender | server | at build time | static HTML on a CDN |
| Streaming SSR | server | per request, in chunks | Suspense boundaries stream in as they resolve |
| RSC | server (or build) | request or build | components that never ship JS to the client |
| PPR | build + request | both | static shell prerendered, dynamic holes resumed per request |

RSC needs a framework or bundler integration: Next.js App Router, React Router (RSC mode), Waku, Parcel, or Vite's RSC plugin. The React APIs below are the same everywhere.

## Server Components

Server Components (the default in an RSC app) run only on the server, at request time or at build time. They can be `async`, can read databases, files, and secrets directly, and their code and dependencies are not sent to the browser. The client receives the rendered result (an RSC payload), not the component code.

```tsx
// app/notes/[id]/page.tsx: a Server Component (no directive needed)
import db from '@/lib/db';
import { marked } from 'marked';           // 35 KB library stays on the server

export default async function NotePage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const note = await db.notes.find(id);    // direct DB access, no API layer
  return (
    <article>
      <h1>{note.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: marked(note.body) }} />  {/* sanitize untrusted markdown */}
      <LikeButton noteId={id} initialLikes={note.likes} />            {/* client island */}
    </article>
  );
}
```

Server Components can't use state, effects, refs, event handlers, browser APIs, or context consumers (`useState`, `useEffect`, `onClick`, `window`, `useContext`).

## 'use client'

`'use client'` at the top of a file marks a **boundary**: this module and everything it imports become client code (rendered on the server for the HTML, then hydrated in the browser). Use it for interactivity, state, effects, and browser APIs.

```tsx
'use client';                                   // must be the first statement

import { useState } from 'react';

export function LikeButton({ noteId, initialLikes }: { noteId: string; initialLikes: number }) {
  const [likes, setLikes] = useState(initialLikes);
  return <button onClick={() => setLikes(likes + 1)}>♥ {likes}</button>;
}
```

- The directive marks the entry point into client code. Files imported by a client component don't need their own directive.
- Props crossing the boundary (server → client) must be serializable: primitives, plain objects and arrays, `Date`, `Map`, `Set`, `BigInt`, typed arrays, `FormData`, promises, JSX, and Server Functions. Not allowed: regular functions, class instances, and symbols (except global ones).
- Server Components can be passed as children to client components. The server renders them, and the client only places the result:

```tsx
// server
<ClientTabs>
  <ServerRenderedPanel />       {/* still a Server Component */}
</ClientTabs>
```

- Keep client boundaries as deep (leaf-level) as possible: a `'use client'` at the top of the page sends the whole page as JS.
- Third-party components that use hooks but lack `'use client'` need a small re-export wrapper: `'use client'; export { Carousel } from 'acme-carousel';`.

## 'server-only' and secrets

```ts
import 'server-only';   // build error if this module is ever imported from client code
export async function getSecretStuff() {
  return fetch(url, { headers: { Authorization: `Bearer ${process.env.API_KEY}` } });
}
```

Never pass secrets or whole DB rows as props to client components: props are serialized into the HTML and the RSC payload. Map them to a minimal DTO first. React's experimental `taintObjectReference` and `taintUniqueValue` add a guard against this.

## Server Functions ('use server')

`'use server'` marks async functions that the client can call over the network. React and the framework turn each one into an RPC endpoint. When used as a form `action` or inside a transition, they're called **Server Actions**.

```ts
// app/actions.ts
'use server';                                   // every export becomes a Server Function

import { z } from 'zod';

const Note = z.object({ title: z.string().min(1).max(200) });

export async function createNote(prev: unknown, formData: FormData) {
  const session = await auth();                         // 1. authenticate: these are PUBLIC endpoints
  if (!session) return { error: 'Unauthorized' };

  const parsed = Note.safeParse({ title: formData.get('title') }); // 2. validate: input is untrusted
  if (!parsed.success) return { error: parsed.error.issues[0].message };

  if (!(await canCreate(session.user))) return { error: 'Forbidden' }; // 3. authorize
  await db.notes.create({ ...parsed.data, ownerId: session.user.id });
  return { ok: true };
}
```

```tsx
// client component using it
'use client';
import { useActionState } from 'react';
import { createNote } from './actions';

export function NewNote() {
  const [state, action, pending] = useActionState(createNote, null);
  return (
    <form action={action}>                         {/* works before JS loads */}
      <input name="title" />
      <button disabled={pending}>Create</button>
      {state?.error && <p role="alert">{state.error}</p>}
    </form>
  );
}
```

```tsx
// inline Server Function inside a Server Component (closure values are encrypted by Next.js)
export default function Page() {
  async function publish(formData: FormData) {
    'use server';
    await db.posts.publish(String(formData.get('id')));
  }
  return <form action={publish}><input type="hidden" name="id" value="1" /><button>Publish</button></form>;
}

// call outside of forms
<button onClick={() => startTransition(async () => { await likeNote(id); })}>Like</button>
```

Security notes:

- Treat every Server Function as a public HTTP endpoint: authenticate, validate, and authorize inside it, every time. A hidden input or a missing button stops no one.
- Arguments arrive deserialized from the client. Never trust IDs or roles sent in them.
- Keep React and your framework patched. In December 2025, critical RSC vulnerabilities (remote code execution, DoS, source exposure) were fixed in 19.0.1/19.1.2/19.2.1 and the later patches.
- `'use server'` is not a marker for server components. A Server Component needs no directive.

## Data, caching and streaming in RSC

```tsx
import { cache, cacheSignal } from 'react';

// cache(): dedupe the same call across components during ONE server request
export const getUser = cache(async (id: string) => {
  return db.users.find(id, { signal: cacheSignal() ?? undefined }); // aborted when the render ends (19.2)
});

// Layout and Page both call getUser(id) → a single DB query per request
```

- Fetch in parallel: start promises first, then await them. Sequential `await`s create waterfalls.
- Pass unawaited promises to client components and read them with `use()` under `<Suspense>` to stream them.
- `cache` only works in Server Components. On the client, use a data library.

```tsx
export default async function Dashboard() {
  const userP = getUser();
  const statsP = getStats();                 // both requests start now
  const [user, stats] = await Promise.all([userP, statsP]);
  return …;
}
```

Framework-level caching (`'use cache'`, `revalidateTag`, ISR) is framework-specific; see [Ecosystem & Tooling](18-ecosystem-tooling.md) for Next.js.

## Server rendering APIs (react-dom)

Frameworks call these for you. You use them directly only in a custom SSR setup.

```tsx
import { renderToPipeableStream } from 'react-dom/server';      // Node streams (preferred in Node)

app.get('/', (req, res) => {
  const { pipe, abort } = renderToPipeableStream(<App />, {
    bootstrapScripts: ['/client.js'],
    onShellReady() { res.setHeader('content-type', 'text/html'); pipe(res); }, // stream as soon as the shell is ready
    onShellError() { res.statusCode = 500; res.send('<h1>Error</h1>'); },
    onError(err) { console.error(err); },
  });
  setTimeout(abort, 10_000);                 // give up on slow Suspense boundaries
});
```

| API | Use |
| --- | --- |
| `renderToPipeableStream` | Node streaming SSR |
| `renderToReadableStream` | Web Streams SSR (Deno, Bun, edge, workers) |
| `prerender` / `prerenderToNodeStream` (`react-dom/static`) | SSG: waits for all data |
| `resume` / `resumeToPipeableStream` (19.2) | PPR: fill in the postponed parts of a prerender |
| `resumeAndPrerender` (19.2) | PPR to static HTML |
| `renderToString` | legacy, no streaming or Suspense data; avoid |

### Hydration

`hydrateRoot` attaches React to server HTML. The first client render must produce the same output as the server, otherwise you get a hydration mismatch.

```tsx
// mismatch sources: Date.now(), Math.random(), window checks, locale formatting, invalid HTML nesting (<p><div/></p>)
<time suppressHydrationWarning>{new Date().toLocaleTimeString()}</time>  // one level, text only

// client-only value: render a placeholder on the server, then update after mount
const [mounted, setMounted] = useState(false);
useEffect(() => setMounted(true), []);
return mounted ? <LocalTime /> : <span>–</span>;
```

### browser() (React 19.3)

Opts a subtree out of server rendering. On the server it suspends, so the nearest Suspense fallback goes into the HTML. In the browser it doesn't suspend, and the content renders after hydration. Unlike the `mounted` trick above, it can be called conditionally.

```tsx
import { use } from 'react';
import { browser } from 'react-dom';

function TimeZone({ defaultValue }: { defaultValue?: string }) {
  if (defaultValue) return <p>{defaultValue}</p>;
  use(browser());                                      // server: fallback; client: continues
  return <p>{Intl.DateTimeFormat().resolvedOptions().timeZone}</p>;
}

<Suspense fallback={<p>…</p>}><TimeZone /></Suspense>
```

<!-- nav -->
---

← [React: Suspense, Concurrency & Errors](10-suspense-concurrent.md) · [Index](README.md) · [React: Styling](12-styling.md) →
<!-- nav -->
