# React: Suspense, Concurrency & Errors

## Concurrent rendering

Since React 18, rendering can be **interrupted**. React can prepare a new UI in the background (a transition), pause it to handle urgent input like typing, and throw away work that has gone stale. Two kinds of updates:

- **urgent**: typing, clicking, pressing. They must show immediately.
- transition: navigating, filtering a big list, loading a tab. They can wait and shouldn't block input.

Concurrent features only switch on when you use them: transitions, `useDeferredValue`, Suspense.

## Suspense

`<Suspense>` shows a fallback while anything inside it is "suspended": waiting for code (`lazy`), data (`use(promise)`, Suspense-enabled libraries), or, inside a `<ViewTransition>`, stylesheets, fonts, and images.

```tsx
import { Suspense } from 'react';

<Suspense fallback={<PageSkeleton />}>
  <Header />
  <Suspense fallback={<FeedSkeleton />}>     {/* nested: reveals in stages */}
    <Feed />
  </Suspense>
</Suspense>
```

- The nearest boundary above the suspended component shows its fallback. All siblings inside the same boundary appear together.
- Effects don't run, and state isn't kept for a tree that suspends on its first mount.
- When already visible content suspends because of an update in a transition, React keeps showing the old UI instead of the fallback. Outside a transition, the fallback replaces it.
- Resetting with a key (`<Suspense key={query}>`) shows the fallback again for new content.
- On the server, Suspense boundaries stream HTML: the fallback is sent first and the content follows when ready. React 19.2+ batches the reveal of close boundaries.

## use(promise)

`use` reads the value of a promise. The component suspends until the promise resolves, and the nearest error boundary catches a rejection.

```tsx
import { use, Suspense } from 'react';

function Comments({ commentsPromise }: { commentsPromise: Promise<Comment[]> }) {
  const comments = use(commentsPromise);       // suspends until resolved
  return comments.map((c) => <p key={c.id}>{c.text}</p>);
}

// The promise must be created OUTSIDE render (or cached). A new promise every render → suspends forever.
function Page({ id }: { id: string }) {
  const commentsPromise = fetchComments(id);   // BAD in a client component: new promise every render
}
```

Where good promises come from:

- a Server Component passing a promise down to a client component (it isn't awaited on the server, so streaming starts sooner),
- a router loader,
- a cache (`cache()` in RSC, or a module-level `Map`),
- libraries with Suspense support: TanStack Query `useSuspenseQuery`, SWR `suspense: true`, Relay, Apollo.

```tsx
// server component: start fetching, don't await, stream the result into a client component
export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const comments = getComments(id);                    // Promise, not awaited
  return (
    <Suspense fallback={<p>Loading comments…</p>}>
      <Comments commentsPromise={comments} />          {/* 'use client' component with use() */}
    </Suspense>
  );
}
```

`use` can't be called inside `try/catch`. Handle errors with an error boundary, or turn the rejection into a value: `promise.catch(() => fallbackValue)`.

## useTransition and startTransition

Marks a state update as non-urgent. React keeps the current UI interactive and renders the new state in the background. `isPending` lets you show inline feedback.

```tsx
import { useState, useTransition } from 'react';

function Tabs() {
  const [tab, setTab] = useState('about');
  const [isPending, startTransition] = useTransition();

  function selectTab(next: string) {
    startTransition(() => {
      setTab(next);                       // slow tab render won't freeze clicks
    });
  }

  return (
    <>
      <TabButton onClick={() => selectTab('posts')}>Posts</TabButton>
      <div style={{ opacity: isPending ? 0.6 : 1 }}>
        {tab === 'posts' ? <PostsTab /> : <AboutTab />}
      </div>
    </>
  );
}
```

```tsx
// async transitions (React 19): "Actions". isPending stays true for the whole async function.
startTransition(async () => {
  const error = await updateName(name);
  // GOTCHA: state set after an await is NOT inside the transition; wrap it again
  startTransition(() => {
    if (error) setError(error);
    else redirect('/profile');
  });
});
```

- `startTransition` can be imported directly from `react` when you don't need `isPending` (outside components, in stores).
- Text inputs can't be controlled by transition updates: the input state must update urgently. Use `useDeferredValue` instead.
- Each transition is independent. Since 19.3, a slow transition no longer holds back unrelated ones.
- `addTransitionType('nav-forward')` (19.3) inside `startTransition` tags *why* the transition happened, so `<ViewTransition>` can pick a matching animation (see below).

## useDeferredValue

Defers re-rendering of a value. Urgent UI (the input) updates first. The expensive part re-renders with the deferred value in the background and can be interrupted by the next keystroke.

```tsx
import { useDeferredValue, useState, memo } from 'react';

function Search() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);   // lags behind query while it's busy
  const isStale = query !== deferredQuery;

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <div style={{ opacity: isStale ? 0.5 : 1 }}>
        <SlowList text={deferredQuery} />          {/* must be memo'd (or compiled) to skip urgent renders */}
      </div>
    </>
  );
}

const SlowList = memo(function SlowList({ text }: { text: string }) { … });

// initial value (React 19): render '' first, then the real value in the background
const value = useDeferredValue(query, '');
```

Combined with Suspense: when `deferredQuery` makes `SearchResults` suspend, the user keeps seeing the previous results instead of a fallback.

Debounce or throttle vs. defer: debouncing waits a fixed delay, while deferring adapts to device speed and can be interrupted. Debounce network requests; defer rendering.

## Activity (React 19.2)

Hides part of the UI without unmounting it. Hidden children are hidden with `display: none`, their effects are cleaned up, and updates are deferred to idle time, but state and DOM are kept. When shown again, effects re-run and the previous state is there.

```tsx
import { Activity } from 'react';

<Activity mode={tab === 'inbox' ? 'visible' : 'hidden'}>
  <Inbox />                 {/* keeps scroll position, drafts, form input */}
</Activity>
<Activity mode={tab === 'settings' ? 'visible' : 'hidden'}>
  <Settings />              {/* pre-renders in the background at low priority */}
</Activity>
```

Uses: tabs and back navigation that keep state, and pre-rendering the likely next screen (including its data and code, since hidden trees can suspend without showing a fallback).

## ViewTransition (React 19.3)

Animates UI changes with the browser's View Transition API. It only animates updates made inside a transition (`startTransition`, a Suspense reveal, `useDeferredValue`), never urgent updates. The default animation is a cross-fade.

```tsx
import { ViewTransition, startTransition, addTransitionType } from 'react';

// enter/exit: element added or removed
{show && (
  <ViewTransition>
    <Panel />
  </ViewTransition>
)}

// shared element: same name removed in one place and added in another → morph
<ViewTransition name={`photo-${id}`}>
  <img src={src} />
</ViewTransition>

// direction-aware animations driven by transition types
function next() {
  startTransition(() => {
    addTransitionType('next');
    setSlide((s) => s + 1);
  });
}

<ViewTransition
  enter={{ next: 'slide-from-right', previous: 'slide-from-left', default: 'none' }}
  exit={{ next: 'slide-to-left', previous: 'slide-to-right', default: 'none' }}
>
  <Slide key={slide} />
</ViewTransition>
```

```css
/* classes passed to enter/exit/update/share become view-transition classes */
::view-transition-new(.slide-from-right) { animation: 300ms ease-out both slide-in-right; }
::view-transition-old(.slide-to-left)    { animation: 300ms ease-in both slide-out-left; }
```

- Props: `name`, `default`, `enter`, `exit`, `update`, `share` (a class string, a `{ [type]: class }` map, or `'none'`), plus `onEnter`/`onExit`/`onUpdate`/`onShare` callbacks for animating with the Web Animations API.
- Wrapping `<Suspense>` animates the fallback → content swap. Recommended: `<ViewTransition update="auto" default="none">`.
- DOM only for now; React Native support is in progress.

## Error boundaries

An error boundary catches errors thrown while rendering, in lifecycle methods, and in the constructors of its children, and shows a fallback UI. It does not catch errors in event handlers, in async code (`setTimeout`, unawaited promises), in SSR, or in itself.

There's still no hook for this. It has to be a class, or the `react-error-boundary` package:

```tsx
class ErrorBoundary extends React.Component<
  { fallback: React.ReactNode; children: React.ReactNode },
  { hasError: boolean }
> {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };              // render the fallback on the next render
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    logError(error, info.componentStack);   // side effects: reporting
  }

  render() {
    return this.state.hasError ? this.props.fallback : this.props.children;
  }
}
```

```tsx
import { ErrorBoundary } from 'react-error-boundary';

<ErrorBoundary
  fallbackRender={({ error, resetErrorBoundary }) => (
    <div role="alert">
      <p>Something went wrong: {error.message}</p>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  )}
  resetKeys={[userId]}                     // auto-reset when these change
  onError={(error, info) => report(error)}
>
  <Suspense fallback={<Spinner />}>
    <Profile userId={userId} />
  </Suspense>
</ErrorBoundary>
```

```tsx
// errors in event handlers / async code: catch them yourself, or push them into a boundary
const { showBoundary } = useErrorBoundary();   // react-error-boundary
fetchData().catch(showBoundary);
```

- Place boundaries at route level, around widgets that can fail independently, and around Suspense boundaries that load data.
- Errors thrown inside transitions or Actions (including a rejected `use(promise)`) propagate to the nearest error boundary.
- Log centrally with the `createRoot` options `onCaughtError` and `onUncaughtError`.

<!-- nav -->
---

← [React: Performance](09-performance.md) · [Index](README.md) · [React: Server Components & Server Functions](11-server-components.md) →
<!-- nav -->
