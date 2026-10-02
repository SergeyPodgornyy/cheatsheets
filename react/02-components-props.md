# React: Components & Props

## Props

Props are the arguments of a component. They are **read-only**: a component never changes its own props, it asks the parent to pass new ones (usually through a callback prop).

```tsx
type AvatarProps = {
  name: string;
  size?: number;                 // optional
  onClick?: () => void;          // callback prop: child → parent communication
};

// destructure in the signature; default values replace removed defaultProps
function Avatar({ name, size = 48, onClick }: AvatarProps) {
  return <img src={`/a/${name}.png`} width={size} alt={name} onClick={onClick} />;
}

<Avatar name="bob" />                   // size = 48
<Avatar name="bob" size={64} />         // non-string values go in {}
<Avatar name="bob" size={undefined} />  // size = 48 (undefined triggers the default)
<Avatar name="bob" size={null} />       // TS error; at runtime null does NOT trigger the default
```

```tsx
// forwarding: spread the rest onto the underlying element
type ButtonProps = React.ComponentProps<'button'> & { variant?: 'primary' | 'ghost' | 'danger' };

function Button({ variant = 'primary', className, ...rest }: ButtonProps) {
  return <button className={`btn btn-${variant} ${className ?? ''}`} {...rest} />;
}

<Button type="submit" disabled>Save</Button>  // type, disabled, children all forwarded
```

```tsx
// boolean shorthand
<input disabled />          // same as disabled={true}
<Modal open={isOpen} />
```

`key` and `ref` are special. `key` is never passed down as a prop. Since React 19, `ref` is passed to function components as a regular prop (see [Refs & DOM](06-refs-dom.md)).

## children

Whatever goes between the tags arrives as the `children` prop. This is the main way to compose components.

```tsx
function Card({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <section className="card">
      <h2>{title}</h2>
      {children}
    </section>
  );
}

<Card title="Profile">
  <Avatar name="bob" />
  <p>Bio…</p>
</Card>
```

```tsx
// "slots": any prop can hold JSX, not just children
function Layout({ header, sidebar, children }: {
  header: React.ReactNode; sidebar: React.ReactNode; children: React.ReactNode;
}) {
  return <>{header}<aside>{sidebar}</aside><main>{children}</main></>;
}

<Layout header={<NavBar />} sidebar={<Menu />}>content</Layout>
```

Passing JSX as children also helps performance: when `Layout` re-renders, `children` is the same element object from the parent, so React skips it.

`React.Children` and `cloneElement` still exist but are legacy and fragile. Prefer explicit props, render props, or context.

## Render props

A prop that is a function returning JSX. The child decides when and with what data to render; the parent decides what.

```tsx
function DataList<T>({ items, renderItem }: {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
}) {
  return <ul>{items.map((it, i) => <li key={i}>{renderItem(it, i)}</li>)}</ul>;
}

<DataList items={users} renderItem={(u) => <b>{u.name}</b>} />
```

Custom hooks have replaced most render-prop use cases. The pattern is still common in libraries (headless UI, virtualization, forms).

## Conditional rendering

```tsx
function Item({ name, isPacked }: { name: string; isPacked: boolean }) {
  // 1. early return
  if (!name) return null;                 // null renders nothing

  return (
    <li>
      {/* 2. ternary for either/or */}
      {isPacked ? <del>{name}</del> : name}

      {/* 3. && for "render or nothing" */}
      {isPacked && ' ✅'}
    </li>
  );
}
```

```tsx
// GOTCHA: && with a number renders the number
{messages.length && <Badge />}        // renders "0" when empty
{messages.length > 0 && <Badge />}    // OK
{!!messages.length && <Badge />}      // OK

// many branches: use a variable or a lookup map instead of nested ternaries
const icons = { success: <Check />, error: <Cross />, loading: <Spinner /> } as const;
return icons[status];
```

## Lists and keys

```tsx
const people = [
  { id: 'a1', name: 'Ada', role: 'dev' },
  { id: 'b2', name: 'Bob', role: 'pm' },
];

function People() {
  return (
    <ul>
      {people
        .filter((p) => p.role === 'dev')
        .map((p) => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

A **key** tells React which array item each component belongs to, so state stays attached to the right item when the list is reordered, inserted into, or filtered.

- Keys must be unique among siblings (not globally) and stable across renders.
- Use IDs from your data. Generate IDs when the item is created, not during render.
- Array index as a key is OK only for static lists that never reorder or filter. Otherwise inputs and state jump between rows.
- `key={Math.random()}` remounts every item on every render: state is lost and it's slow.

```tsx
// key on a fragment when one item renders several elements
import { Fragment } from 'react';

{posts.map((p) => (
  <Fragment key={p.id}>
    <dt>{p.title}</dt>
    <dd>{p.body}</dd>
  </Fragment>
))}
```

```tsx
// key outside lists: change the key to reset a component's state
<ChatInput key={contactId} />   // new contact → fresh input, old draft gone
```

## Fragments

```tsx
<>
  <td>A</td>
  <td>B</td>
</>
// long form when you need a key (or a ref, React 19.3+):
<Fragment key={id}>…</Fragment>
```

## Composition over inheritance

React has no component inheritance. Reuse UI through composition (children and slots), logic through custom hooks, and variants through props.

```tsx
// specialization: a "special case" component renders the general one with fixed props
function DangerButton(props: Omit<ButtonProps, 'variant'>) {
  return <Button {...props} variant="danger" />;
}
```

## Containers and presentational components

A common split is:

- **presentational**: props in, JSX out, no data fetching. Easy to test and to put in Storybook.
- **container**: fetches data and holds state, then renders presentational children.

With hooks, the container is often just a custom hook (`useUser(id)`). With Server Components, it's often an `async` server component (see [Server Components](11-server-components.md)).

## Class components (legacy)

You'll still see class components in older code. New code uses functions and hooks. The one thing that still requires a class is an error boundary (see [Suspense & Errors](10-suspense-concurrent.md)).

```tsx
class Counter extends React.Component<{ step: number }, { count: number }> {
  state = { count: 0 };

  componentDidMount() { /* ≈ useEffect(..., []) */ }
  componentDidUpdate(prevProps: { step: number }) { /* ≈ useEffect with deps */ }
  componentWillUnmount() { /* ≈ effect cleanup */ }

  render() {
    return (
      <button onClick={() => this.setState((s) => ({ count: s.count + this.props.step }))}>
        {this.state.count}
      </button>
    );
  }
}
```

Removed in React 19: `propTypes` and `defaultProps` on function components (use TypeScript and default parameters), string refs, legacy context (`contextTypes`), `createFactory`, and `react-test-renderer/shallow`.

<!-- nav -->
---

← [React: Basics](01-basics.md) · [Index](README.md) · [React: State](03-state.md) →
<!-- nav -->
