# React: TypeScript

```console
$ npm i -D typescript @types/react @types/react-dom
```

```jsonc
// tsconfig.json (relevant bits)
{
  "compilerOptions": {
    "jsx": "react-jsx",            // modern transform: no `import React` needed
    "strict": true,
    "moduleResolution": "bundler",
    "verbatimModuleSyntax": true,  // forces `import type` for type-only imports
    "noUncheckedIndexedAccess": true
  }
}
```

## Typing props

```tsx
type Props = {
  title: string;
  count?: number;                              // optional
  status: 'idle' | 'loading' | 'error';        // literal union instead of a string
  items: readonly Item[];
  onSelect: (id: string) => void;              // callback
  children: React.ReactNode;                   // anything renderable
  icon?: React.ReactElement;                   // exactly one JSX element
  renderRow?: (item: Item) => React.ReactNode; // render prop
  style?: React.CSSProperties;
};

function Panel({ title, count = 0, ...rest }: Props) { … }
```

`type` and `interface` both work for props. `React.FC` is no longer needed (it used to add implicit `children`); type the props parameter directly.

| Type | Accepts |
| --- | --- |
| `React.ReactNode` | JSX, string, number, boolean, null, undefined, arrays, portals, promises (19) |
| `React.ReactElement` / `React.JSX.Element` | a single JSX element only |
| `React.ComponentType<P>` | a component (function or class) that takes `P` |
| `React.ElementType` | a tag name (`'a'`) or a component, for polymorphic `as` props |
| `React.PropsWithChildren<P>` | `P & { children?: ReactNode }` |

## Reusing HTML and component props

```tsx
type InputProps = React.ComponentProps<'input'>;                 // every <input> prop, including ref (19)
type ButtonProps = React.ComponentPropsWithoutRef<'button'>;     // without ref
type MyBtnProps = React.ComponentProps<typeof MyButton>;         // props of an existing component

type Props = Omit<React.ComponentProps<'input'>, 'size'> & { size: 'sm' | 'lg' }; // override one prop
```

## Discriminated union props

Make invalid prop combinations impossible to express:

```tsx
type Props =
  | { variant: 'link'; href: string; onClick?: never }
  | { variant: 'button'; onClick: () => void; href?: never };

function Action(props: Props) {
  if (props.variant === 'link') return <a href={props.href}>Go</a>;   // narrowed: href is string
  return <button onClick={props.onClick}>Go</button>;
}

<Action variant="link" href="/x" />
<Action variant="link" onClick={f} />    // error
```

## Generic components

```tsx
type SelectProps<T> = {
  options: T[];
  value: T | null;
  onChange: (value: T) => void;
  getLabel: (option: T) => string;
  getKey: (option: T) => string | number;
};

function Select<T>({ options, value, onChange, getLabel, getKey }: SelectProps<T>) {
  return (
    <ul>
      {options.map((o) => (
        <li key={getKey(o)} aria-selected={o === value} onClick={() => onChange(o)}>{getLabel(o)}</li>
      ))}
    </ul>
  );
}

<Select options={users} value={u} onChange={setU} getLabel={(u) => u.name} getKey={(u) => u.id} />
// T is inferred as User; onChange gets (value: User) => void

// arrow-function generics in .tsx need a trailing comma: const Select = <T,>(props: SelectProps<T>) => …
```

## Polymorphic "as" prop

```tsx
type TextProps<C extends React.ElementType> = {
  as?: C;
  children: React.ReactNode;
} & Omit<React.ComponentProps<C>, 'as' | 'children'>;

function Text<C extends React.ElementType = 'span'>({ as, ...rest }: TextProps<C>) {
  const Component = as ?? 'span';
  return <Component {...rest} />;
}

<Text as="a" href="/home">Home</Text>           // href allowed because of 'a'
<Text as="label" htmlFor="x">Name</Text>
```

## Hooks

```tsx
const [count, setCount] = useState(0);                       // inferred number
const [user, setUser] = useState<User | null>(null);         // union needs a type argument
const [items, setItems] = useState<Item[]>([]);              // [] would be never[]

const inputRef = useRef<HTMLInputElement>(null);             // RefObject<HTMLInputElement | null> (19)
const timer = useRef<number | undefined>(undefined);         // React 19: useRef requires an argument

const [state, dispatch] = useReducer(reducer, initialState); // types come from the reducer signature

const value = useMemo(() => compute(a), [a]);                // inferred
const onClick = useCallback((e: React.MouseEvent<HTMLButtonElement>) => {}, []);

const Ctx = createContext<Theme | null>(null);
```

```tsx
// setters as props
type Props = { setCount: React.Dispatch<React.SetStateAction<number>> };
// simpler and more flexible: onCountChange: (n: number) => void
```

## Events

```tsx
function handleChange(e: React.ChangeEvent<HTMLInputElement>) { e.target.value; }
function handleSubmit(e: React.SubmitEvent<HTMLFormElement>) { e.preventDefault(); }
function handleKey(e: React.KeyboardEvent) { if (e.key === 'Enter') …; }

type Props = { onClick: React.MouseEventHandler<HTMLButtonElement> };   // handler types
// lazy approach: write the handler inline and hover it to see the inferred type
```

## Refs and the ref prop

```tsx
// React 19: ref is a normal prop
function Input({ ref, ...props }: React.ComponentProps<'input'>) {
  return <input ref={ref} {...props} />;
}

// custom imperative handle
type DialogHandle = { open(): void; close(): void };
function Dialog({ ref }: { ref?: React.Ref<DialogHandle> }) {
  useImperativeHandle(ref, () => ({ open() {}, close() {} }), []);
  return null;
}
```

## Utility patterns

```tsx
// exhaustive switch
function assertNever(x: never): never { throw new Error(`Unexpected: ${JSON.stringify(x)}`); }

// typed "as const" maps for variants
const sizes = { sm: 'px-2 text-sm', md: 'px-4', lg: 'px-6 text-lg' } as const;
type Size = keyof typeof sizes;          // 'sm' | 'md' | 'lg'

// satisfies: check a config against a type while keeping literal inference
const routes = { home: '/', user: '/users/:id' } satisfies Record<string, `/${string}`>;
```

Global JSX types live in the `React.JSX` namespace (`React.JSX.IntrinsicElements`). The global `JSX` namespace was removed from @types/react 19.

Runtime validation: TypeScript types disappear at runtime. Parse untrusted data (API responses, forms, URL params, `localStorage`) with Zod, Valibot, or ArkType, and infer the TS type from the schema.

```ts
const User = z.object({ id: z.string(), name: z.string(), role: z.enum(['admin', 'user']) });
type User = z.infer<typeof User>;
const user = User.parse(await res.json());    // throws on unexpected shape
```

<!-- nav -->
---

← [React: Data Fetching & State Libraries](14-data-state-libs.md) · [Index](README.md) · [React: Testing](16-testing.md) →
<!-- nav -->
