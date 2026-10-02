# React: Patterns, Accessibility & Security

## Controlled vs uncontrolled components

The same distinction as for inputs applies to your own components. **Uncontrolled**: the component owns its state (`defaultOpen`). **Controlled**: the parent owns it (`open` + `onOpenChange`). Good components support both:

```tsx
function useControllableState<T>({ value, defaultValue, onChange }: {
  value?: T; defaultValue: T; onChange?: (v: T) => void;
}) {
  const [internal, setInternal] = useState(defaultValue);
  const isControlled = value !== undefined;
  const current = isControlled ? value : internal;

  const setValue = (next: T) => {
    if (!isControlled) setInternal(next);
    onChange?.(next);
  };
  return [current, setValue] as const;
}

function Disclosure({ open, defaultOpen = false, onOpenChange, children }: Props) {
  const [isOpen, setOpen] = useControllableState({ value: open, defaultValue: defaultOpen, onChange: onOpenChange });
  return <details open={isOpen} onToggle={(e) => setOpen(e.currentTarget.open)}>{children}</details>;
}

<Disclosure defaultOpen />                                // uncontrolled
<Disclosure open={open} onOpenChange={setOpen} />         // controlled
```

## Compound components

Components that share implicit state through context and read as one unit (like `<select>` and `<option>`).

```tsx
const TabsCtx = createContext<{ active: string; setActive(v: string): void } | null>(null);
const useTabs = () => { const c = use(TabsCtx); if (!c) throw new Error('Inside <Tabs> only'); return c; };

export function Tabs({ defaultValue, children }: { defaultValue: string; children: React.ReactNode }) {
  const [active, setActive] = useState(defaultValue);
  return <TabsCtx value={{ active, setActive }}>{children}</TabsCtx>;
}

Tabs.List = function List({ children }: { children: React.ReactNode }) {
  return <div role="tablist">{children}</div>;
};

Tabs.Tab = function Tab({ value, children }: { value: string; children: React.ReactNode }) {
  const { active, setActive } = useTabs();
  return (
    <button role="tab" aria-selected={active === value} onClick={() => setActive(value)}>
      {children}
    </button>
  );
};

Tabs.Panel = function Panel({ value, children }: { value: string; children: React.ReactNode }) {
  const { active } = useTabs();
  return active === value ? <div role="tabpanel">{children}</div> : null;
};

<Tabs defaultValue="a">
  <Tabs.List>
    <Tabs.Tab value="a">Account</Tabs.Tab>
    <Tabs.Tab value="b">Billing</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel value="a">…</Tabs.Panel>
  <Tabs.Panel value="b">…</Tabs.Panel>
</Tabs>
```

Dot notation (`Tabs.Tab`) doesn't work across the RSC client boundary. In RSC apps, export `TabsList`, `TabsTab`, and so on as named exports.

## Headless components and hooks

Split behavior (state, keyboard, ARIA) from markup and styles. Expose the behavior as a hook or as unstyled components, and let consumers render whatever they want. This is how Radix, React Aria, Downshift, and TanStack Table work.

```tsx
function useToggle(initial = false) {
  const [on, setOn] = useState(initial);
  return {
    on,
    toggle: () => setOn((o) => !o),
    buttonProps: { 'aria-pressed': on, onClick: () => setOn((o) => !o) },  // "prop getter"
  };
}

const { on, buttonProps } = useToggle();
<button {...buttonProps} className={on ? 'on' : ''}>Bold</button>
```

## Higher-order components (legacy pattern)

A function that takes a component and returns an enhanced one: `withAuth(Page)`. Hooks have mostly replaced them. HOCs hide where props come from and are awkward to type. You'll still see them in older libraries (`connect`, `withRouter`).

## Project structure

Group by feature, not by file type, once the app grows past a handful of screens:

```text
src/
├── app/                 # app shell, providers, router setup
├── features/
│   ├── auth/
│   │   ├── components/  # LoginForm.tsx
│   │   ├── hooks/       # useSession.ts
│   │   ├── api.ts       # queries/mutations
│   │   └── index.ts     # public surface of the feature
│   └── cart/
├── components/ui/       # shared, generic UI (Button, Dialog)
├── lib/                 # utils, api client, config
└── test/
```

- One component per file for anything non-trivial. Name the file after the component.
- Colocate tests, styles, and stories next to the component.
- Avoid deep barrel files (`index.ts` re-exporting everything). They slow down dev servers and tests and get in the way of tree-shaking.
- Use path aliases (`@/components/...`) through `tsconfig` `paths` plus the bundler config.

## Accessibility (a11y)

- Use semantic HTML first: `<button>` for actions, `<a href>` for navigation, `<label>`, `<nav>`, `<main>`, headings in order. A `<div onClick>` isn't focusable or keyboard-accessible.
- Every input needs a label (`<label htmlFor>` with `useId`, or `aria-label`). Images need `alt` (`alt=""` for decorative ones).
- Keyboard: everything must work with Tab, Enter, Space, Escape, and arrow keys where expected. Don't remove focus outlines without a replacement (`:focus-visible`).
- Focus management: move focus into dialogs and back to the trigger when they close (native `<dialog>` + `showModal()` does this), and move focus to the new page heading after client navigation.
- Announce async changes with live regions: `<div role="status" aria-live="polite">` for toasts, `role="alert"` for errors.
- ARIA is a last resort: "no ARIA is better than bad ARIA". Headless libraries (React Aria, Radix) get complex widgets right.
- Respect `prefers-reduced-motion` in animations.
- Tooling: `eslint-plugin-jsx-a11y`, axe DevTools, Lighthouse, and testing with a screen reader (VoiceOver, NVDA).

```tsx
function IconButton({ label, icon, ...props }: { label: string; icon: React.ReactNode } & React.ComponentProps<'button'>) {
  return (
    <button aria-label={label} {...props}>
      <span aria-hidden="true">{icon}</span>
    </button>
  );
}
```

## Security

- XSS: JSX escapes interpolated values. The risky spots are `dangerouslySetInnerHTML` (sanitize with DOMPurify), user-controlled `href`/`src` (allow only `http(s):` and `mailto:` URLs), and spreading untrusted objects as props (`{...userData}` can set `dangerouslySetInnerHTML`).
- Never put secrets in client code. Anything bundled for the browser is public: `VITE_*` and `NEXT_PUBLIC_*` env vars, props passed to client components, and the RSC payload. Keep API keys on the server (Server Functions, route handlers, a backend).
- Server Functions are public endpoints: authenticate, authorize, and validate with a schema inside each one (see [Server Components](11-server-components.md)).
- Auth tokens: prefer `HttpOnly`, `Secure`, `SameSite` cookies over `localStorage` (which any XSS can read). Protect cookie-based mutations against CSRF (SameSite, Origin checks; frameworks handle this for Server Actions).
- CSP: set a Content-Security-Policy with nonces for inline scripts (`renderToPipeableStream(…, { nonce })`). React 19.3 supports Trusted Types (`require-trusted-types-for 'script'`), passing `TrustedHTML` through without turning it into a string.
- Dependencies: run `npm audit`, use Dependabot or Renovate, pin versions in a lockfile, and patch React and framework security releases quickly.
- Open redirects: validate `?redirect=` targets against an allow-list of relative paths.

```tsx
function safeHref(url: string) {
  try {
    const u = new URL(url, window.location.origin);
    return ['http:', 'https:', 'mailto:'].includes(u.protocol) ? u.href : '#';
  } catch {
    return '#';
  }
}
```

<!-- nav -->
---

← [React: Testing](16-testing.md) · [Index](README.md) · [React: Ecosystem & Tooling](18-ecosystem-tooling.md) →
<!-- nav -->
