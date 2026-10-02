# React: Basics

## What React is

React is a **library** for building user interfaces out of **components**: functions that take data (props) and return a description of the UI. React is **declarative**: you describe what the screen should look like for the current state, and React figures out which DOM changes are needed. Think of it as `UI = f(state)`.

React itself only knows about components, state, and rendering. The DOM part lives in `react-dom` (web), and `react-native` covers mobile. Routing, data fetching, and bundling come from the ecosystem or from a framework.

Current version: React 19.3 (September 2026). Examples here use TypeScript (`.tsx`), which is the default for new projects.

## Creating a project

Create React App is deprecated (sunset in February 2025). Start with a framework for full apps, or with Vite for a client-only SPA.

```console
$ npm create vite@latest my-app -- --template react-ts   # SPA: Vite + React + TS
$ npx create-next-app@latest                               # full-stack: Next.js (App Router, RSC)
$ npx create-react-router@latest                           # React Router framework mode
$ npx create-expo-app@latest                               # React Native via Expo
```

```console
$ cd my-app && npm install
$ npm run dev        # dev server with Fast Refresh (state survives edits)
$ npm run build      # production bundle in dist/
$ npm run preview    # serve the production build locally
```

Typical Vite layout:

```text
my-app/
├── index.html          # entry HTML; loads /src/main.tsx as a module
├── public/             # copied as-is, served from /
├── src/
│   ├── main.tsx        # createRoot(...).render(<App />)
│   ├── App.tsx
│   └── assets/         # imported assets get hashed file names
├── vite.config.ts
└── tsconfig.json
```

## Rendering the root

```tsx
// src/main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';
import './index.css';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

```tsx
// Root error hooks (React 19): log errors in one place
createRoot(container, {
  onUncaughtError: (error, info) => report(error, info.componentStack),
  onCaughtError: (error, info) => report(error),   // caught by an error boundary
  onRecoverableError: (error) => console.warn(error), // e.g. hydration mismatch React recovered from
});

// Server-rendered HTML: attach to existing markup instead of replacing it
import { hydrateRoot } from 'react-dom/client';
hydrateRoot(document, <App />);

root.unmount(); // tear down the tree (rare; micro-frontends, tests)
```

`ReactDOM.render` and `ReactDOM.hydrate` were removed in React 19. Use `createRoot` and `hydrateRoot`.

### StrictMode

`<StrictMode>` only does something in development. It helps catch impure code:

- renders each component twice (impure render logic shows up as inconsistent output)
- runs effects mount → cleanup → mount once (a missing cleanup shows up as duplicated subscriptions or requests)
- calls ref callbacks twice, and warns about deprecated APIs

```tsx
useEffect(() => {
  console.log('connect'); // dev + StrictMode: connect, disconnect, connect
  return () => console.log('disconnect');
}, []);
```

## JSX

JSX is syntax sugar that compiles to function calls. `<h1 className="t">Hi</h1>` becomes `jsx('h1', { className: 't', children: 'Hi' })`. With the modern JSX transform you no longer need `import React` in every file.

```tsx
const name = 'Sergey';
const user = { avatar: '/me.png', isAdmin: true };

const element = (
  <div className="card">                     {/* class → className */}
    <label htmlFor="email">Email</label>     {/* for → htmlFor */}
    <input id="email" tabIndex={0} />        {/* camelCase attributes */}
    <img src={user.avatar} alt="" />         {/* every tag must be closed */}
    <p>Hello, {name.toUpperCase()}!</p>       {/* {} holds any JS expression */}
    <p style={{ color: 'red', fontSize: 14 }}>styled</p> {/* style is an object; 14 → "14px" */}
    <button aria-label="Close" data-id="7">×</button> {/* aria-* and data-* stay kebab-case */}
  </div>
);
```

Rules:

- Return a single root. Wrap siblings in a fragment `<>...</>` when you don't want an extra DOM node.
- `{}` takes expressions only. `if`, `for`, and `switch` are statements, so use them before the `return`, or switch to `&&`, ternaries, or `.map()`.
- Lowercase tags (`<div>`) are DOM elements. Capitalized tags (`<Button>`) are components.
- Comments go inside braces: `{/* comment */}`.

```tsx
// what renders for each value type
<div>{'text'}</div>      // text
<div>{42}</div>          // 42
<div>{0}</div>           // 0  ← GOTCHA: zero renders
<div>{true}</div>        // nothing (also false, null, undefined)
<div>{[1, 2, 3]}</div>   // 123 (arrays are flattened)
<div>{{ a: 1 }}</div>    // Error: Objects are not valid as a React child
```

### Escaping and raw HTML

JSX escapes every string you embed, so `{userInput}` can't inject markup. Raw HTML needs an explicit opt-in:

```tsx
// DANGER: only with trusted or sanitized HTML (e.g. DOMPurify.sanitize(html))
<div dangerouslySetInnerHTML={{ __html: sanitizedHtml }} />

// href with user input can still run code: javascript:alert(1)
// React 19 blocks javascript: URLs in src/href/action, but validate URLs anyway
<a href={isSafeUrl(url) ? url : '#'}>link</a>
```

## Components

A component is a function that starts with a capital letter and returns JSX (or `null` to render nothing).

```tsx
function Greeting() {
  return <h1>Hello!</h1>;
}

export default function App() {
  return (
    <main>
      <Greeting />
      <Greeting />
    </main>
  );
}
```

```tsx
// GOTCHA: never define a component inside another component.
// Each render creates a NEW function → React sees a new component type
// → unmounts the old subtree and loses its state (and focus) every render.
function Parent() {
  function Child() { return <input />; } // BAD
  return <Child />;
}
```

## Purity

Rendering must be **pure**: same props, state, and context give the same JSX, and nothing outside the component changes. Side effects go in event handlers, or in effects as a last resort.

```tsx
let guests = 0;
function Cup() {
  guests += 1;                       // BAD: mutates outside state during render
  return <p>Cup for guest #{guests}</p>;
}

function Cup({ guest }: { guest: number }) {
  return <p>Cup for guest #{guest}</p>; // GOOD: output depends only on props
}
```

Local mutation is fine: creating an array and pushing to it inside render doesn't break purity.

## Render and commit

React updates the screen in three steps:

1. **Trigger**: the initial `root.render()`, or a state update (`setX`) in this component or an ancestor.
2. **Render**: React calls your components and works out what changed. It's recursive: a parent re-render re-renders all of its children by default.
3. **Commit**: React applies the minimal DOM changes, then runs layout effects, the browser paints, then passive effects (`useEffect`) run.

Rendering is not the same as updating the DOM. A component can re-render and produce identical output, and then React touches nothing.

## React DevTools

Browser extension with two tabs. Components lets you inspect props, state, and hooks, and edit them live. Profiler records commits and shows why each component rendered. The React Compiler marks auto-memoized components with a "Memo ✨" badge. In Chrome's Performance panel, React 19.2+ adds Scheduler and Components tracks.

<!-- nav -->
---

← [Home](../README.md) · [Index](README.md) · [React: Components & Props](02-components-props.md) →
<!-- nav -->
