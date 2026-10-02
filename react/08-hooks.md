# React: Hooks & Custom Hooks

## Rules of hooks

1. Call hooks only at the top level of a component or a custom hook: not in conditions, loops, nested functions, `try/catch`, or after an early `return`.
2. Call hooks only from React functions: components or custom hooks, never from regular functions or class components.

React identifies each hook by its **call order**. A conditional hook shifts the order and mixes up state between hooks.

```tsx
function Bad({ id }: { id?: string }) {
  if (!id) return null;
  const [x, setX] = useState(0);        // BAD: skipped on some renders
}

function Good({ id }: { id?: string }) {
  const [x, setX] = useState(0);        // always called
  if (!id) return null;
}
```

`use()` is the exception: it's not a hook in this sense and can be called inside `if` and after early returns (still not inside `try/catch`).

Enforced by `eslint-plugin-react-hooks` (`rules-of-hooks`, `exhaustive-deps`, plus the React Compiler rules).

## Built-in hooks reference

| Hook | Purpose | See |
| --- | --- | --- |
| `useState` | local state | [State](03-state.md) |
| `useReducer` | state with reducer logic | [State](03-state.md) |
| `useContext` / `use(Context)` | read context | [Context](07-context.md) |
| `useRef` | mutable box, DOM refs | [Refs & DOM](06-refs-dom.md) |
| `useImperativeHandle` | customize the ref API | [Refs & DOM](06-refs-dom.md) |
| `useEffect` | sync with external systems | [Effects](05-effects.md) |
| `useLayoutEffect` | measure before paint | [Effects](05-effects.md) |
| `useInsertionEffect` | CSS-in-JS style injection | [Effects](05-effects.md) |
| `useEffectEvent` | non-reactive logic in effects (19.2) | [Effects](05-effects.md) |
| `useMemo` / `useCallback` | memoize values / functions | [Performance](09-performance.md) |
| `useTransition` / `useDeferredValue` | non-blocking updates | [Suspense & Concurrency](10-suspense-concurrent.md) |
| `use` | read a promise or context | [Suspense & Concurrency](10-suspense-concurrent.md) |
| `useActionState` | action result + pending | [Events & Forms](04-events-forms.md) |
| `useOptimistic` | optimistic UI | [Events & Forms](04-events-forms.md) |
| `useFormStatus` (react-dom) | parent form's pending state | [Events & Forms](04-events-forms.md) |
| `useId` | unique IDs, SSR-safe | below |
| `useSyncExternalStore` | subscribe to an external store | below |
| `useDebugValue` | label a custom hook in DevTools | below |

## useId

Generates a unique ID that is identical on the server and the client, so hydration doesn't mismatch. Use it for accessibility attributes, never for list keys.

```tsx
function Field({ label }: { label: string }) {
  const id = useId();                      // "_r_1_" (format changed in 19.2; don't depend on it)
  return (
    <>
      <label htmlFor={id}>{label}</label>
      <input id={id} aria-describedby={`${id}-hint`} />
      <p id={`${id}-hint`}>Required</p>
    </>
  );
}
// several roots on one page: createRoot(el, { identifierPrefix: 'app2-' })
```

## useSyncExternalStore

Subscribes to a store that lives outside React (browser APIs, third-party stores, your own module state) without tearing during concurrent rendering.

```tsx
import { useSyncExternalStore } from 'react';

function subscribe(callback: () => void) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

export function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,                      // stable function: define it outside the component
    () => navigator.onLine,         // getSnapshot (client)
    () => true,                     // getServerSnapshot (SSR + hydration)
  );
}
```

```tsx
// a minimal store
let state = { count: 0 };
const listeners = new Set<() => void>();

export const store = {
  get: () => state,                              // must return the SAME reference if nothing changed
  set(next: typeof state) { state = next; listeners.forEach((l) => l()); },
  subscribe(l: () => void) { listeners.add(l); return () => listeners.delete(l); },
};

const count = useSyncExternalStore(store.subscribe, () => store.get().count);
// GOTCHA: getSnapshot returning a new object every call ({...state}) → infinite re-render loop
```

## useDebugValue

```tsx
function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, () => navigator.onLine);
  useDebugValue(isOnline ? 'Online' : 'Offline');   // shown next to the hook in DevTools
  return isOnline;
}
```

## Custom hooks

A custom hook is a function whose name starts with `use` and which calls other hooks. It shares stateful logic, not state: every component that calls it gets its own independent state.

```tsx
export function useLocalStorage<T>(key: string, initial: T) {
  const [value, setValue] = useState<T>(() => {
    try {
      const raw = localStorage.getItem(key);
      return raw === null ? initial : (JSON.parse(raw) as T);
    } catch {
      return initial;                 // corrupted JSON, or storage unavailable (private mode)
    }
  });

  useEffect(() => {
    try { localStorage.setItem(key, JSON.stringify(value)); } catch { /* quota exceeded */ }
  }, [key, value]);

  return [value, setValue] as const;  // as const → typed tuple, not (T | Dispatch)[]
}

const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');
```

```tsx
export function useDebouncedValue<T>(value: T, delay = 300) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);
  return debounced;
}
```

```tsx
export function useMediaQuery(query: string) {
  const subscribe = useCallback((cb: () => void) => {   // a new subscribe function → resubscribe
    const mql = window.matchMedia(query);
    mql.addEventListener('change', cb);
    return () => mql.removeEventListener('change', cb);
  }, [query]);

  return useSyncExternalStore(
    subscribe,
    () => window.matchMedia(query).matches,
    () => false,
  );
}
```

```tsx
// a hook that accepts an event handler: keep the handler out of the dependencies
export function useInterval(callback: () => void, ms: number | null) {
  const onTick = useEffectEvent(callback);
  useEffect(() => {
    if (ms === null) return;                 // null pauses the interval
    const id = setInterval(() => onTick(), ms);
    return () => clearInterval(id);
  }, [ms]);
}
```

Guidelines:

- Name hooks after what they do (`useChatRoom`, `useOnlineStatus`), not after lifecycle (`useMount`, `useEffectOnce`). Lifecycle-style hooks hide dependency bugs.
- A function that calls no hooks is a regular function: don't give it the `use` prefix.
- Return an object for many values, a tuple `as const` for a `[value, setter]` pair.
- Don't fetch data in hand-written custom hooks for anything non-trivial. Use TanStack Query, SWR, or your framework's loaders.

Ready-made collections: `usehooks-ts`, `@uidotdev/usehooks`, `react-use`, `ahooks`.

<!-- nav -->
---

← [React: Context](07-context.md) · [Index](README.md) · [React: Performance](09-performance.md) →
<!-- nav -->
