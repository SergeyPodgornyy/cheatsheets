# React: State

## useState

State is a component's memory: data that persists between renders, where changing it triggers a re-render. A regular local variable resets on every render, and changing it doesn't re-render anything.

```tsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);         // [current value, setter]
  const [name, setName] = useState<string | null>(null); // explicit type when the initial value is narrower

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

Each component instance gets its own state. Two `<Counter />` elements count independently. State belongs to the position in the tree, not to the function.

### State is a snapshot

`setX` doesn't change the variable in the current render. It schedules a re-render with the new value.

```tsx
const [count, setCount] = useState(0);

function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
  console.log(count);  // 0: still the value from this render
}                      // next render: 1, not 3

function handleClickFixed() {
  setCount((c) => c + 1);  // updater function: receives the pending value
  setCount((c) => c + 1);
  setCount((c) => c + 1);  // next render: 3
}

// even after a timeout, a closure sees the render it was created in
setTimeout(() => alert(count), 3000); // shows the count at click time
```

Use the updater form when the new state depends on the previous one, especially when it can be called several times or from async code.

### Batching

React batches every `setX` call inside the same event, timeout, or promise into one re-render (automatic batching since React 18). To force a synchronous DOM update, use `flushSync` (rarely needed; see [Refs & DOM](06-refs-dom.md)).

### Lazy initialization

```tsx
const [todos, setTodos] = useState(createInitialTodos());  // BAD: runs on every render, result ignored
const [todos, setTodos] = useState(createInitialTodos);    // GOOD: called only on the first render
const [todos, setTodos] = useState(() => JSON.parse(localStorage.getItem('todos') ?? '[]'));
```

### Same value skips the render

If the new value is identical to the current one (`Object.is`), React skips re-rendering the children. That's why mutating state doesn't work: the reference stays the same.

## Updating objects and arrays

Treat state as **immutable**. Create a new object or array instead of changing the existing one.

```tsx
const [user, setUser] = useState({ name: 'Ada', address: { city: 'London' } });

user.name = 'Bob';                  // BAD: mutation, no re-render
setUser(user);                      // BAD: same reference → skipped

setUser({ ...user, name: 'Bob' });  // GOOD: shallow copy + override
setUser((u) => ({ ...u, address: { ...u.address, city: 'Paris' } })); // nested: copy every level
```

```tsx
const [items, setItems] = useState<Item[]>([]);

setItems([...items, newItem]);                            // add to the end
setItems([newItem, ...items]);                            // add to the start
setItems(items.filter((i) => i.id !== id));               // remove
setItems(items.map((i) => (i.id === id ? { ...i, done: !i.done } : i))); // update one
setItems([...items.slice(0, idx), newItem, ...items.slice(idx)]);        // insert at index
setItems(items.toSorted((a, b) => a.price - b.price));    // sort (ES2023; .sort() mutates)
setItems(items.toReversed());                             // reverse (.reverse() mutates)
setItems(items.with(idx, newItem));                       // replace at index (ES2023)
```

| Mutates (avoid) | Returns new array (use) |
| --- | --- |
| `push`, `unshift` | `[...arr, x]`, `[x, ...arr]` |
| `pop`, `shift`, `splice` | `filter`, `slice`, `toSpliced` |
| `arr[i] = x` | `map`, `with` |
| `sort`, `reverse` | `toSorted`, `toReversed` |

Deeply nested updates get verbose. Either flatten (normalize) the state, or use Immer (`useImmer`, or `produce` inside an updater) to write mutable-looking code that produces immutable updates.

```tsx
import { useImmer } from 'use-immer';

const [user, updateUser] = useImmer({ address: { city: 'London' } });
updateUser((draft) => { draft.address.city = 'Paris'; }); // draft mutation → new object
```

## Structuring state

- Group values that always change together (`{ x, y }` instead of two states).
- Avoid contradictions: one `status: 'idle' | 'sending' | 'sent'` beats `isSending` + `isSent`.
- Avoid redundant state: compute it during render instead of syncing it.
- Avoid duplication: store the `selectedId`, not a copy of the selected object.
- Avoid deep nesting: normalize into `{ byId: {...}, ids: [...] }`.

```tsx
// BAD: fullName is redundant and goes stale unless you sync it with an effect
const [first, setFirst] = useState('');
const [last, setLast] = useState('');
const [fullName, setFullName] = useState('');

// GOOD: derive it during render
const fullName = `${first} ${last}`;
```

```tsx
// GOTCHA: copying a prop into state "freezes" it; later prop changes are ignored
function Message({ color }: { color: string }) {
  const [c] = useState(color);        // only the first color is used
  const c2 = color;                   // use the prop directly instead
}
// if freezing is intentional, name it that way: initialColor
```

## Lifting state up

When two components need the same state, move it to their closest common parent and pass it down along with a setter callback. The child becomes **controlled** by the parent.

```tsx
function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);   // single source of truth
  return (
    <>
      <Panel title="About" isActive={activeIndex === 0} onShow={() => setActiveIndex(0)} />
      <Panel title="Etymology" isActive={activeIndex === 1} onShow={() => setActiveIndex(1)} />
    </>
  );
}

function Panel({ title, isActive, onShow }: { title: string; isActive: boolean; onShow: () => void }) {
  return (
    <section>
      <h3>{title}</h3>
      {isActive ? <p>…</p> : <button onClick={onShow}>Show</button>}
    </section>
  );
}
```

When lifting means passing props through many layers ("prop drilling"), first try composition (pass JSX as `children`). If that's not enough, use [Context](07-context.md) or a store.

## Preserving and resetting state

React keeps a component's state as long as the same component type renders at the same position in the tree.

```tsx
// same position, same type → state is KEPT when isFancy toggles
{isFancy ? <Counter isFancy /> : <Counter />}

// different type at the same position → state is RESET (whole subtree remounts)
{isPaused ? <p>Paused</p> : <Counter />}

// reset on purpose: different keys
{isPlayerA ? <Counter key="A" /> : <Counter key="B" />}

// different positions → independent state
{isPlayerA && <Counter />}
{!isPlayerA && <Counter />}
```

To keep the state of a component that is currently hidden, use `<Activity mode="hidden">` (see [Suspense & Concurrency](10-suspense-concurrent.md)), lift the state up, or just hide it with CSS.

## Adjusting state when a prop changes

Usually you can derive the value or reset it with a key. In the rare case you need to adjust only part of the state, do it during render, guarded by a stored previous value. Avoid an effect here: it renders stale UI first, then renders again.

```tsx
function List({ items }: { items: Item[] }) {
  const [selection, setSelection] = useState<Item | null>(null);
  const [prevItems, setPrevItems] = useState(items);

  if (items !== prevItems) {   // only setState of THIS component during render, with a guard
    setPrevItems(items);
    setSelection(null);
  }
  // simpler still: store selectedId and derive the selected item from items
}
```

## useReducer

Moves the update logic into one pure function. Event handlers dispatch actions that describe what happened; the reducer decides how state changes. Worth it once several handlers update the same state in different ways.

```tsx
import { useReducer } from 'react';

type Todo = { id: number; text: string; done: boolean };
type Action =
  | { type: 'added'; id: number; text: string }
  | { type: 'toggled'; id: number }
  | { type: 'deleted'; id: number };

function todosReducer(todos: Todo[], action: Action): Todo[] {
  switch (action.type) {
    case 'added':
      return [...todos, { id: action.id, text: action.text, done: false }];
    case 'toggled':
      return todos.map((t) => (t.id === action.id ? { ...t, done: !t.done } : t));
    case 'deleted':
      return todos.filter((t) => t.id !== action.id);
    default: {
      const _exhaustive: never = action;   // compile error if a case is missing
      throw new Error('Unknown action');
    }
  }
}

function TodoApp() {
  const [todos, dispatch] = useReducer(todosReducer, []);
  // lazy init: useReducer(reducer, arg, init) → init(arg) runs once

  return (
    <button onClick={() => dispatch({ type: 'added', id: Date.now(), text: 'New' })}>
      Add ({todos.length})
    </button>
  );
}
```

- Reducers must be pure: no requests, timers, or mutation. In StrictMode they run twice.
- `dispatch` has a stable identity, so it's safe to leave out of dependency arrays and to pass down through context.
- Pair `useReducer` with context to get a small app-wide store (see [Context](07-context.md)).

| | `useState` | `useReducer` |
| --- | --- | --- |
| Code size | less up front | reducer + action types |
| Readability | simple updates | complex, related updates in one place |
| Debugging | harder to trace who set what | log every action in the reducer |
| Testing | through the component | reducer is a plain function |

<!-- nav -->
---

← [React: Components & Props](02-components-props.md) · [Index](README.md) · [React: Events & Forms](04-events-forms.md) →
<!-- nav -->
