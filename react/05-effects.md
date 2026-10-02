# React: Effects

## What effects are for

Effects **synchronize** a component with an external system: network, browser APIs, timers, subscriptions, third-party widgets, analytics. They run after the commit, once the screen has updated.

If no external system is involved, you probably don't need an effect. Event-specific logic belongs in event handlers; derived values belong in render.

## useEffect

```tsx
import { useEffect, useState } from 'react';

function ChatRoom({ roomId }: { roomId: string }) {
  useEffect(() => {
    const conn = createConnection(roomId);   // setup
    conn.connect();
    return () => conn.disconnect();          // cleanup: before the next run AND on unmount
  }, [roomId]);                              // re-sync whenever roomId changes

  return <h1>Room {roomId}</h1>;
}
```

| Dependencies | Runs |
| --- | --- |
| `useEffect(fn)` | after every render |
| `useEffect(fn, [])` | after mount only (cleanup on unmount) |
| `useEffect(fn, [a, b])` | after mount and whenever `a` or `b` changes (`Object.is`) |

- Dependencies aren't something you choose. Every reactive value the effect reads (props, state, and anything computed from them in the component body) must be listed. The `react-hooks/exhaustive-deps` lint rule enforces this.
- To remove a dependency, show that the effect doesn't need it: move objects and functions inside the effect, move constants outside the component, use an updater `setX(x => ...)`, or use `useEffectEvent`.
- In development StrictMode runs setup → cleanup → setup. If that breaks something, the fix is a proper cleanup, not "run once" refs.

```tsx
// GOTCHA: object/function deps are recreated every render → the effect runs every render
function Chat({ roomId }: { roomId: string }) {
  const options = { serverUrl, roomId };          // new object every render
  useEffect(() => {
    const c = createConnection(options);
    c.connect();
    return () => c.disconnect();
  }, [options]);                                  // BAD: reconnects on every render

  useEffect(() => {
    const c = createConnection({ serverUrl, roomId }); // GOOD: build it inside
    c.connect();
    return () => c.disconnect();
  }, [roomId]);
}
```

### Common cleanups

```tsx
useEffect(() => {
  const id = setInterval(tick, 1000);
  return () => clearInterval(id);
}, []);

useEffect(() => {
  const onResize = () => setWidth(window.innerWidth);
  window.addEventListener('resize', onResize);
  return () => window.removeEventListener('resize', onResize);
}, []);

useEffect(() => {
  const dialog = ref.current!;
  dialog.showModal();
  return () => dialog.close();
}, []);

useEffect(() => {
  const node = ref.current!;
  node.style.opacity = '1';                 // trigger an animation
  return () => { node.style.opacity = '0'; };
}, []);
```

## Fetching data in effects

Doable, but you have to handle races, loading, errors, and caching yourself. Prefer a framework loader, a Server Component, `use()` with a cached promise, or TanStack Query (see [Data Fetching & Stores](14-data-state-libs.md)).

```tsx
function Profile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const controller = new AbortController();
    setUser(null);

    fetch(`/api/users/${userId}`, { signal: controller.signal })
      .then((r) => {
        if (!r.ok) throw new Error(`HTTP ${r.status}`);
        return r.json();
      })
      .then(setUser)
      .catch((e) => {
        if (e.name !== 'AbortError') setError(e);
      });

    return () => controller.abort();   // stale response for an old userId is dropped
  }, [userId]);
}
```

Without the abort (or an `ignore` flag), quickly switching `userId` from 1 to 2 can show user 1's data if that response arrives last: a **race condition**.

The effect callback itself can't be `async` (it must return a cleanup or nothing). Define an async function inside and call it.

## You might not need an effect

| Instead of an effect that... | Do this |
| --- | --- |
| computes a value from props/state | compute it during render (`const total = items.reduce(...)`) |
| caches an expensive computation | `useMemo` (or rely on the React Compiler) |
| resets all state when a prop changes | `key` on the component |
| adjusts some state when a prop changes | derive it, or set state during render (see [State](03-state.md)) |
| reacts to a user action (submit, buy) | put the logic in the event handler |
| notifies the parent about a state change | call the parent's callback in the same handler |
| chains state updates (A changes → set B → set C) | compute everything in one event handler |
| subscribes to an external store | `useSyncExternalStore` |
| initializes the app once | module-level code, or a module-level `didInit` guard |

```tsx
// BAD: an extra render, and briefly stale UI
const [visible, setVisible] = useState<Todo[]>([]);
useEffect(() => setVisible(todos.filter(matches(filter))), [todos, filter]);

// GOOD
const visible = todos.filter(matches(filter));
```

```tsx
// BAD: the "purchase" fires because a value changed, not because the user clicked
useEffect(() => {
  if (product.isInCart) showNotification(`Added ${product.name}`);
}, [product]);

// GOOD: event-specific logic in the event handler
function handleBuy() {
  addToCart(product);
  showNotification(`Added ${product.name}`);
}
```

## useEffectEvent

Since React 19.2. Pulls non-reactive logic out of an effect. The function always sees the latest props and state, but it isn't a dependency, so changing those values doesn't re-run the effect.

```tsx
import { useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId, theme }: { roomId: string; theme: string }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);   // reads the latest theme
  });

  useEffect(() => {
    const conn = createConnection(roomId);
    conn.on('connected', () => onConnected());
    conn.connect();
    return () => conn.disconnect();
  }, [roomId]);                               // a theme change does NOT reconnect
}
```

Rules (the lint rule checks them):

- call it only from inside effects (or other effect events), never during render, and don't pass it to other components or hooks
- declare it in the same component or hook as the effect that uses it
- never list it in a dependency array
- don't use it to silence the linter; use it only for logic that behaves like an "event" fired from an effect

## useLayoutEffect

Same API, but it runs synchronously after the DOM is updated and before the browser paints. Use it to measure layout and re-render before the user sees anything, such as tooltip positioning. It blocks paint, so use `useEffect` unless you see flicker.

```tsx
function Tooltip({ targetRect, children }: Props) {
  const ref = useRef<HTMLDivElement>(null);
  const [height, setHeight] = useState(0);

  useLayoutEffect(() => {
    setHeight(ref.current!.getBoundingClientRect().height); // re-renders before paint
  }, []);

  const top = targetRect.top - height < 0 ? targetRect.bottom : targetRect.top - height;
  return <div ref={ref} style={{ position: 'absolute', top }}>{children}</div>;
}
```

On the server, layout effects don't run (no layout). Measure on the client only, or render a fallback.

## useInsertionEffect

Runs before layout effects. It exists for CSS-in-JS libraries to inject `<style>` tags before anyone reads layout. You won't need it in app code.

## Effect timing summary

```text
render (pure) → commit DOM → useInsertionEffect → ref callbacks / useLayoutEffect
              → browser paints → useEffect
```

An effect triggered by a click (a discrete input) is flushed synchronously before the next input, so the result can be visible before paint.

<!-- nav -->
---

← [React: Events & Forms](04-events-forms.md) · [Index](README.md) · [React: Refs & DOM](06-refs-dom.md) →
<!-- nav -->
