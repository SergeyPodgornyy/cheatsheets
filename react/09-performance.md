# React: Performance

## Why components re-render

A component re-renders when:

1. its state changes,
2. its parent re-renders (by default all children re-render, even with identical props),
3. a context it reads changes,
4. an external store it subscribes to changes.

Props changing is not a trigger by itself. Props only change because the parent re-rendered. A re-render is usually cheap. Measure before optimizing.

## Measure first

- React DevTools Profiler: record an interaction and see which components rendered, how long they took, and why ("Highlight updates when components render" in the settings).
- Chrome Performance panel: React 19.2+ adds Scheduler and Components tracks (in dev builds, and in profiling builds via `react-dom/profiling`).
- `<Profiler>` for programmatic timing:

```tsx
import { Profiler } from 'react';

<Profiler id="Sidebar" onRender={(id, phase, actualDuration, baseDuration) => {
  // phase: 'mount' | 'update' | 'nested-update'; actualDuration: ms spent rendering this commit
  metrics.push({ id, phase, actualDuration });
}}>
  <Sidebar />
</Profiler>
```

Always profile a production build (`npm run build && npm run preview`). Dev mode is much slower and double-renders in StrictMode.

## React Compiler

React Compiler (v1.0, October 2025) is a build-time tool that memoizes components and hooks automatically: values, callbacks, and JSX. In most code that makes manual `memo`, `useMemo`, and `useCallback` unnecessary. It works with React 17+ (19 is best), and only compiles code that follows the Rules of React.

```console
$ npm install -D babel-plugin-react-compiler@latest   # Babel plugin (Next.js, Babel setups)
```

```ts
// vite.config.ts: Vite 8 + @vitejs/plugin-react 6 (Babel path)
import { defineConfig } from 'vite';
import react, { reactCompilerPreset } from '@vitejs/plugin-react';
import babel from '@rolldown/plugin-babel';           // npm i -D @rolldown/plugin-babel @babel/core

export default defineConfig({
  plugins: [react(), babel({ presets: [reactCompilerPreset()] })],
  // experimental Rust port (oxc): plugins: [react({ compiler: true })]
});
```

```ts
// next.config.ts
const nextConfig = { reactCompiler: true };
export default nextConfig;
```

Expo SDK 54+ enables it by default.

```tsx
// opt a single component/hook out (escape hatch while you fix a rules violation)
function LegacyWidget() {
  'use no memo';
  …
}

// incremental adoption: compile only annotated code
// reactCompilerPreset({ compilationMode: 'annotation' }) + 'use memo' at the top of a function
```

- Lint with `eslint-plugin-react-hooks` (the `recommended` preset includes compiler rules like `set-state-in-render`, `set-state-in-effect`, `refs`, `purity`, `immutability`). Code the compiler can't optimize is skipped, not broken.
- Existing `useMemo`/`useCallback` can stay. Remove them only with tests, since that can change what gets compiled.
- Keep `useMemo`/`useCallback` as escape hatches, for example when a memoized value is used as an effect dependency and must keep a stable identity.

## memo

`memo` skips re-rendering a component when its props are shallowly equal (`Object.is` per prop) to the previous ones. It's an optimization, never a guarantee.

```tsx
import { memo } from 'react';

const Chart = memo(function Chart({ data, onSelect }: ChartProps) {
  return <ExpensiveSvg data={data} onSelect={onSelect} />;
});

// custom comparison (rare; must compare every prop, including functions)
const Row = memo(RowImpl, (prev, next) => prev.item.version === next.item.version);
```

```tsx
// GOTCHA: new objects/functions every render defeat memo
function Parent() {
  return <Chart
    data={{ points }}                 // new object → Chart re-renders
    onSelect={() => select()}         // new function → Chart re-renders
  />;
}
```

## useMemo and useCallback

```tsx
import { useMemo, useCallback } from 'react';

function TodoList({ todos, tab, theme }: Props) {
  // cache an expensive calculation between renders; recomputed when todos/tab change
  const visible = useMemo(() => filterTodos(todos, tab), [todos, tab]);

  // cache a function identity (= useMemo(() => fn, deps))
  const handleSubmit = useCallback((order: Order) => {
    post(`/product/${productId}/buy`, order);
  }, [productId]);

  return <div className={theme}><List items={visible} onSubmit={handleSubmit} /></div>;
}
```

Use them when (without the compiler):

- the calculation is noticeably slow (more than about 1 ms; check with `console.time`),
- the value or function is passed to a `memo` component,
- the value is a dependency of another hook (effect, memo).

`useMemo` is a performance hint: React may throw the cache away. Don't rely on it for correctness, and never put side effects in it. In StrictMode the calculation runs twice.

## Structural fixes (often better than memo)

```tsx
// 1. move state down: only the part that changes re-renders
function App() {
  return (
    <>
      <SearchBox />          {/* owns its own query state */}
      <ExpensiveTree />      {/* no longer re-renders on every keystroke */}
    </>
  );
}

// 2. lift content up: pass the expensive part as children
function ColorPicker({ children }: { children: React.ReactNode }) {
  const [color, setColor] = useState('red');
  return <div style={{ color }}><input value={color} onChange={(e) => setColor(e.target.value)} />{children}</div>;
}
<ColorPicker><ExpensiveTree /></ColorPicker>   // children element is created by App → not re-rendered
```

Other structural fixes:

- Keep state as local as possible. Global state that changes often re-renders everything that subscribes to it.
- Use selectors with external stores (`useStore((s) => s.count)`) so components re-render only for the slice they use.
- Make transitions non-blocking: `useTransition`/`useDeferredValue` keep typing responsive while a heavy list renders (see [Suspense & Concurrency](10-suspense-concurrent.md)).

## Code splitting

```tsx
import { lazy, Suspense } from 'react';

const Editor = lazy(() => import('./Editor'));          // must be a default export
const Chart = lazy(() => import('./charts').then((m) => ({ default: m.Chart }))); // named export

function Page({ editing }: { editing: boolean }) {
  return (
    <Suspense fallback={<Spinner />}>
      {editing && <Editor />}                            {/* chunk loads on first render */}
    </Suspense>
  );
}

// preload on intent (hover/focus) to hide latency
const loadEditor = () => import('./Editor');
<button onMouseEnter={loadEditor} onClick={() => setEditing(true)}>Edit</button>
```

Declare `lazy` components at module level. Declaring them inside a component recreates them and resets state on every render. Routers and frameworks split by route automatically.

## Large lists

Rendering 10,000 rows is slow no matter how much you memoize. Virtualize: render only the visible rows.

```tsx
import { useVirtualizer } from '@tanstack/react-virtual';

function BigList({ rows }: { rows: Row[] }) {
  const parentRef = useRef<HTMLDivElement>(null);
  const virtualizer = useVirtualizer({
    count: rows.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 36,
    overscan: 5,
  });

  return (
    <div ref={parentRef} style={{ height: 400, overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize(), position: 'relative' }}>
        {virtualizer.getVirtualItems().map((v) => (
          <div key={v.key} style={{ position: 'absolute', top: 0, transform: `translateY(${v.start}px)`, height: v.size, width: '100%' }}>
            {rows[v.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

Alternatives: `react-window`, `react-virtuoso`. For simple pages, CSS `content-visibility: auto` lets the browser skip off-screen work.

## Other tips

- Use stable, unique keys. Index keys on changing lists cause extra DOM work and bugs.
- Debounce or throttle expensive handlers (search, resize, scroll).
- Bundle size: analyze it (`rollup-plugin-visualizer`, `@next/bundle-analyzer`), prefer ESM packages that tree-shake, and import only what you use.
- Images: lazy-load them (`loading="lazy"`), set `width`/`height` to avoid layout shift, and use `fetchPriority="high"` for the hero image.
- Web Vitals (`web-vitals` package): track LCP, INP, and CLS in production.

<!-- nav -->
---

← [React: Hooks & Custom Hooks](08-hooks.md) · [Index](README.md) · [React: Suspense, Concurrency & Errors](10-suspense-concurrent.md) →
<!-- nav -->
