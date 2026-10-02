# React: Styling

React doesn't prescribe a styling approach. All of these work; the trade-offs are runtime cost, Server Component support, and colocation.

## className and plain CSS

```tsx
import './Button.css';                              // global CSS: class names can collide across files

<button className="btn btn-primary">Save</button>
<button className={`btn ${isActive ? 'active' : ''}`}>Tab</button>
```

```tsx
// conditional classes: clsx (tiny) instead of string juggling
import clsx from 'clsx';

<button className={clsx('btn', { active: isActive, disabled }, size === 'lg' && 'btn-lg')} />
```

## Inline styles

```tsx
<div style={{ backgroundColor: 'teal', marginTop: 8, width: '50%', '--accent': color } as React.CSSProperties} />
// camelCase properties; numbers get "px" except unitless ones (opacity, zIndex, flex, lineHeight)
// CSS variables (--x) need a cast in TS
```

Use inline styles for dynamic values (positions, user-chosen colors) and classes for everything else. Inline styles can't do `:hover`, media queries, or pseudo-elements.

## CSS Modules

Class names are scoped per file at build time (`.title` → `Card_title__x7a2`). Vite and Next.js support CSS Modules with no setup. They add no runtime cost and work in Server Components.

```css
/* Card.module.css */
.card { padding: 1rem; border-radius: 8px; }
.title { font-weight: 600; }
.card:hover .title { color: var(--accent); }
```

```tsx
import styles from './Card.module.css';

<div className={styles.card}>
  <h2 className={styles.title}>Hi</h2>
</div>
```

## Tailwind CSS

Utility classes in markup. The most common choice in new React projects (it's the default in `create-next-app`). v4 is configured in CSS, with no JS config file.

```console
$ npm install tailwindcss @tailwindcss/vite
```

```ts
// vite.config.ts
import tailwindcss from '@tailwindcss/vite';
export default defineConfig({ plugins: [react(), tailwindcss()] });
```

```css
/* src/index.css */
@import "tailwindcss";

@theme {
  --color-brand: oklch(0.65 0.2 250);   /* → bg-brand, text-brand, ... */
}
```

```tsx
<button className="rounded-lg bg-brand px-4 py-2 text-white hover:bg-brand/90 disabled:opacity-50 md:px-6 dark:bg-slate-800">
  Save
</button>
```

```tsx
// merging classes without conflicts (shadcn/ui pattern)
import { twMerge } from 'tailwind-merge';
import clsx, { type ClassValue } from 'clsx';

export const cn = (...inputs: ClassValue[]) => twMerge(clsx(inputs));

cn('px-2 py-1', isLarge && 'px-4');   // "py-1 px-4": the later px wins
```

Write full class names. Tailwind scans source files for strings, so `bg-${color}-500` is never generated. Map values to complete classes instead: `{ red: 'bg-red-500', blue: 'bg-blue-500' }[color]`.

Variants: `cva` (class-variance-authority) or `tailwind-variants` define typed `variant` and `size` props on top of utility classes.

## CSS-in-JS

| Library | Runtime | RSC |
| --- | --- | --- |
| styled-components, Emotion | runtime (styles injected in the browser) | client components only |
| vanilla-extract, Panda CSS, StyleX, Linaria, Pigment CSS | zero-runtime (extracted at build time) | yes |

```tsx
// styled-components
import styled from 'styled-components';

const Button = styled.button<{ $primary?: boolean }>`
  padding: 0.5rem 1rem;
  background: ${(p) => (p.$primary ? 'teal' : 'white')};   /* $ prefix: transient, not forwarded to the DOM */
  &:hover { opacity: 0.9; }
`;
```

Runtime CSS-in-JS costs render time and has trouble with streaming SSR and Server Components. New projects mostly pick CSS Modules, Tailwind, or a zero-runtime library.

## Stylesheets in React 19

`<link rel="stylesheet" precedence="...">` and `<style href precedence>` can be rendered from any component. React deduplicates them, orders them by `precedence`, hoists them into `<head>`, and suspends until they load, so there's no flash of unstyled content.

```tsx
function Card() {
  return (
    <>
      <link rel="stylesheet" href="/card.css" precedence="default" />
      <div className="card">…</div>
    </>
  );
}
```

## Component libraries

- Headless (behavior + accessibility, you style them): Radix UI, React Aria (Adobe), Base UI, Headless UI, Ark UI.
- Copy-paste: shadcn/ui (Radix or Base UI + Tailwind, the code lives in your repo).
- Styled kits: MUI, Mantine, Chakra UI, Ant Design, HeroUI.
- Animation: Motion (formerly Framer Motion), React Spring, AutoAnimate; for simple cases, CSS transitions plus `<ViewTransition>`.
- Icons: lucide-react, react-icons, Heroicons.

<!-- nav -->
---

← [React: Server Components & Server Functions](11-server-components.md) · [Index](README.md) · [React: Routing](13-routing.md) →
<!-- nav -->
