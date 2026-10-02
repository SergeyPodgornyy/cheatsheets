# React: Refs & DOM

## useRef

A ref is a box `{ current: value }` that persists across renders. Changing `ref.current` doesn't re-render. Use it for values that don't affect the output: timer IDs, DOM nodes, previous values, instances of imperative libraries.

```tsx
import { useRef, useState } from 'react';

function Stopwatch() {
  const [start, setStart] = useState<number | null>(null);
  const [now, setNow] = useState<number | null>(null);
  const intervalRef = useRef<number | null>(null);   // survives renders, no re-render

  function handleStart() {
    setStart(Date.now());
    setNow(Date.now());
    clearInterval(intervalRef.current ?? undefined);
    intervalRef.current = window.setInterval(() => setNow(Date.now()), 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current ?? undefined);
  }

  const secondsPassed = start && now ? (now - start) / 1000 : 0;
  return <>{secondsPassed.toFixed(3)} <button onClick={handleStart}>Start</button></>;
}
```

| | ref | state |
| --- | --- | --- |
| Returns | `{ current }` | `[value, setValue]` |
| Change triggers re-render | no | yes |
| Mutable | yes, change `current` directly | no, use the setter |
| Read during render | don't (except lazy init) | yes |

```tsx
// don't read or write refs during render (output becomes unpredictable); use them in handlers and effects
function Bad() {
  const count = useRef(0);
  count.current++;                     // BAD: impure render
  return <p>{count.current}</p>;       // BAD: not reactive
}

// exception: lazy one-time init
const playerRef = useRef<VideoPlayer | null>(null);
if (playerRef.current === null) playerRef.current = new VideoPlayer();
```

## DOM refs

```tsx
function Form() {
  const inputRef = useRef<HTMLInputElement>(null);   // null until commit

  return (
    <>
      <input ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>Focus</button>
    </>
  );
}
```

Common uses: focus, scroll (`scrollIntoView`), measure (`getBoundingClientRect`), media playback, canvas, and integrating non-React widgets (maps, charts, editors).

Don't change the DOM that React manages (removing a node React rendered → crash). Modifying parts React never touches (an empty `<div ref>` for a chart lib) is fine.

## ref as a prop (React 19)

Function components receive `ref` as a regular prop. `forwardRef` is no longer needed and will be deprecated.

```tsx
function MyInput({ ref, ...props }: React.ComponentProps<'input'>) {
  return <input ref={ref} {...props} />;
}

// parent
const ref = useRef<HTMLInputElement>(null);
<MyInput ref={ref} placeholder="Name" />
```

```tsx
// legacy (React ≤ 18): still works in 19
const MyInput = forwardRef<HTMLInputElement, Props>(function MyInput(props, ref) {
  return <input ref={ref} {...props} />;
});
```

## Ref callbacks

Pass a function instead of a ref object. React calls it with the node after mount. Since React 19 it can return a cleanup function, which runs when the node is removed or the callback changes.

```tsx
<div
  ref={(node) => {
    const observer = new ResizeObserver(([entry]) => setSize(entry.contentRect));
    observer.observe(node!);
    return () => observer.disconnect();      // React 19 cleanup
  }}
/>
```

```tsx
// refs to a dynamic list of items: keep a Map in one ref
const itemsRef = useRef(new Map<string, HTMLLIElement>());

{cats.map((cat) => (
  <li key={cat.id} ref={(node) => {
    itemsRef.current.set(cat.id, node!);
    return () => { itemsRef.current.delete(cat.id); };
  }}>{cat.name}</li>
))}

itemsRef.current.get(id)?.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
```

An inline callback is a new function on each render, so React calls cleanup and then setup again on every re-render. Usually harmless; with the React Compiler it's memoized for you.

GOTCHA: since a returned function now means cleanup, TypeScript rejects ref callbacks with an implicit return like `ref={(n) => (inputRef.current = n)}`. Use a block body: `ref={(n) => { inputRef.current = n; }}`.

## useImperativeHandle

Expose a custom, limited API through a ref instead of the whole DOM node.

```tsx
type VideoHandle = { play(): void; pause(): void };

function Video({ ref, src }: { ref: React.Ref<VideoHandle>; src: string }) {
  const videoRef = useRef<HTMLVideoElement>(null);

  useImperativeHandle(ref, () => ({
    play: () => videoRef.current?.play(),
    pause: () => videoRef.current?.pause(),
  }), []);

  return <video ref={videoRef} src={src} />;
}

const player = useRef<VideoHandle>(null);
<Video ref={player} src="/a.mp4" />
player.current?.play();
```

Prefer props (`isPlaying`) when you can express the thing declaratively. Imperative handles are for focus, scroll, and animation triggers.

## Fragment refs (React 19.3)

A ref on `<Fragment>` gives a `FragmentInstance` that works on the group of child DOM nodes without adding a wrapper element.

```tsx
import { Fragment, useEffect, useRef } from 'react';

function Group({ items }: { items: Item[] }) {
  const ref = useRef<React.FragmentInstance>(null);   // DOM methods typed by @types/react-dom

  useEffect(() => {
    const fragment = ref.current!;
    fragment.focus();                                   // first focusable descendant
    const onClick = () => console.log('clicked a child');
    fragment.addEventListener('click', onClick);        // on first-level child elements
    const io = new IntersectionObserver(onVisible);
    fragment.observeUsing(io);                          // observe each child
    return () => {
      fragment.removeEventListener('click', onClick);
      fragment.unobserveUsing(io);
    };
  }, []);

  return <Fragment ref={ref}>{items.map((i) => <Row key={i.id} {...i} />)}</Fragment>;
}
```

Methods: `addEventListener`, `removeEventListener`, `dispatchEvent`, `focus`, `focusLast`, `blur`, `observeUsing`, `unobserveUsing`, `getClientRects`, `getRootNode`, `compareDocumentPosition`, `scrollIntoView`.

## flushSync

Forces React to apply a state update to the DOM synchronously, for example when you need the new node right away. It's expensive and should be rare.

```tsx
import { flushSync } from 'react-dom';

function handleAdd() {
  flushSync(() => {
    setTodos([...todos, newTodo]);
  });
  listRef.current!.lastElementChild!.scrollIntoView();  // the new item is already in the DOM
}
```

## Portals

`createPortal` renders children into a different DOM node (such as `document.body`) while keeping them in the same place in the React tree. Context, state, and event bubbling follow the React tree, not the DOM tree.

```tsx
import { createPortal } from 'react-dom';

function Modal({ open, onClose, children }: Props) {
  if (!open) return null;
  return createPortal(
    <div className="backdrop" onClick={onClose}>
      <div role="dialog" aria-modal="true" onClick={(e) => e.stopPropagation()}>
        {children}
      </div>
    </div>,
    document.body,
  );
}
// a click inside the modal bubbles to Modal's React parents, even though the DOM node is in <body>
```

Use cases: modals, tooltips, dropdowns that escape `overflow: hidden` or z-index stacking, and rendering React into non-React DOM. The native `<dialog>` element and the Popover API (`popover` attribute) cover many of these cases without a portal.

## Document metadata (React 19)

`<title>`, `<meta>`, and `<link>` rendered anywhere in the tree are hoisted into `<head>`. This works on the client, in SSR, and in RSC.

```tsx
function BlogPost({ post }: { post: Post }) {
  return (
    <article>
      <title>{post.title}</title>
      <meta name="author" content={post.author} />
      <link rel="canonical" href={post.url} />
      <h1>{post.title}</h1>
    </article>
  );
}
```

```tsx
// stylesheets with precedence: deduplicated, ordered, and Suspense waits for them to load
<link rel="stylesheet" href="/card.css" precedence="default" />
<style href="card-inline" precedence="default">{`.card{padding:8px}`}</style>

// async scripts: deduplicated, rendered once no matter how many components include them
<script async src="https://maps.example.com/api.js" />
```

### Resource hints

```tsx
import { prefetchDNS, preconnect, preload, preloadModule, preinit, preinitModule } from 'react-dom';

function App() {
  preinit('https://cdn.x.com/widget.js', { as: 'script' });   // load and execute now
  preload('/fonts/inter.woff2', { as: 'font', type: 'font/woff2', crossOrigin: 'anonymous' });
  preload('/hero.webp', { as: 'image', fetchPriority: 'high' });
  preconnect('https://api.x.com');
  prefetchDNS('https://img.x.com');
  return …;
}
```

They can be called during render or in event handlers (for example, preload the next page's assets on hover). In SSR they are emitted as `<link>` tags early in the HTML stream.

<!-- nav -->
---

← [React: Effects](05-effects.md) · [Index](README.md) · [React: Context](07-context.md) →
<!-- nav -->
