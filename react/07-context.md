# React: Context

## Why context

Context passes data deep down the tree without threading props through every level. Typical uses: theme, current user, locale, routing, and app-wide stores (state + dispatch).

Before reaching for it, try passing props explicitly (it makes data flow obvious) or composition (`children`), which often removes the intermediate layers altogether.

## Create, provide, consume

```tsx
import { createContext, use, useContext, useState } from 'react';

type Theme = 'light' | 'dark';
export const ThemeContext = createContext<Theme>('light');   // default: used when there's no provider above

function App() {
  const [theme, setTheme] = useState<Theme>('dark');
  return (
    <ThemeContext value={theme}>          {/* React 19: the context itself is the provider */}
      <Page />
      <button onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}>Toggle</button>
    </ThemeContext>
  );
}

function Button() {
  const theme = useContext(ThemeContext);  // classic hook: nearest provider ABOVE this component
  const same = use(ThemeContext);          // React 19: same result, but allowed after early returns and inside if
  return <button className={theme}>OK</button>;
}
```

- `<Ctx.Provider value>` still works. `<Ctx value>` is the React 19 shorthand, and `Provider` will be deprecated.
- A component reads the closest provider above it. Providers can be nested to override a value for a subtree.
- When the value changes (`Object.is`), every consumer re-renders, even through `memo` boundaries.
- Since React 19.3, Server Components can render `<Ctx value>` directly for a context created in a `'use client'` module.

```tsx
// use() can be called conditionally; useContext can't
function Heading({ children }: { children?: React.ReactNode }) {
  if (!children) return null;
  const level = use(LevelContext);
  return <h1 data-level={level}>{children}</h1>;
}
```

## Context with a safe hook

A `null` default plus a custom hook gives a clear error instead of silent default values when the provider is missing.

```tsx
type Auth = { user: User | null; login(u: User): void; logout(): void };
const AuthContext = createContext<Auth | null>(null);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const value = useMemo<Auth>(() => ({            // stable object: consumers don't re-render on every parent render
    user,
    login: setUser,
    logout: () => setUser(null),
  }), [user]);                                    // with the React Compiler, the useMemo isn't needed

  return <AuthContext value={value}>{children}</AuthContext>;
}

export function useAuth() {
  const ctx = use(AuthContext);
  if (!ctx) throw new Error('useAuth must be used inside <AuthProvider>');
  return ctx;
}
```

## Reducer + context

A small app-wide store with no library. Splitting state and dispatch into two contexts means components that only dispatch don't re-render when the state changes.

```tsx
const TasksContext = createContext<Task[] | null>(null);
const TasksDispatchContext = createContext<React.Dispatch<Action> | null>(null);

export function TasksProvider({ children }: { children: React.ReactNode }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>   {/* dispatch is stable */}
        {children}
      </TasksDispatchContext>
    </TasksContext>
  );
}

export const useTasks = () => use(TasksContext)!;
export const useTasksDispatch = () => use(TasksDispatchContext)!;

function AddTask() {
  const dispatch = useTasksDispatch();          // doesn't re-render when tasks change
  return <button onClick={() => dispatch({ type: 'added', id: nextId++, text: 'x' })}>Add</button>;
}
```

## Performance pitfalls

```tsx
// BAD: a new object every render → every consumer re-renders when App re-renders
<UserContext value={{ user, setUser }}>

// GOOD options
const value = useMemo(() => ({ user, setUser }), [user]); // memoize (or let the Compiler do it)
<UserContext value={user}>  <SetUserContext value={setUser}>  // split fast/slow-changing parts
```

- Put fast-changing values (mouse position, form input) in separate contexts, or in an external store with selectors (Zustand, Redux, `useSyncExternalStore`). Context has no selectors: consumers re-render on any change of the value.
- Keep providers as low as possible. A provider at the root that changes often re-renders a lot of the app.
- `children` passed to a provider component don't re-render just because the provider's state changed. Only the consumers do.

## When to use what

| Need | Tool |
| --- | --- |
| a few levels of props | props |
| layout wrappers passing things down | composition / `children` |
| rarely changing global values (theme, locale, auth) | context |
| frequently updated shared client state | Zustand / Redux Toolkit / Jotai |
| server data (cache, refetch, dedupe) | TanStack Query, framework loaders, RSC |
| URL-shareable state (filters, tabs, page) | search params (router) |

<!-- nav -->
---

← [React: Refs & DOM](06-refs-dom.md) · [Index](README.md) · [React: Hooks & Custom Hooks](08-hooks.md) →
<!-- nav -->
