# React: Events & Forms

## Event handlers

Pass a function, don't call it. Handlers are named `handleX` inside the component and passed down as `onX` props.

```tsx
function Toolbar({ onPlay }: { onPlay: () => void }) {
  function handleClick(e: React.MouseEvent<HTMLButtonElement>) {
    console.log(e.currentTarget.textContent);
    onPlay();
  }

  return (
    <>
      <button onClick={handleClick}>Play</button>
      <button onClick={() => alert('hi')}>Inline</button>
      <button onClick={alert('hi')}>BAD</button>  {/* runs during render! */}
    </>
  );
}
```

React wraps native events into **SyntheticEvents** that behave the same in every browser. `e.nativeEvent` gives you the original. Events are attached once at the root container (delegation), not to each element.

```tsx
<div onClick={() => console.log('div')}>       {/* bubbles: fires second */}
  <button onClick={(e) => {
    e.stopPropagation();                        // stop bubbling to parents
    console.log('button');
  }}>Click</button>
</div>

<form onSubmit={(e) => {
  e.preventDefault();                           // stop the browser's default action (page reload)
}} />

<div onClickCapture={() => {}} />              {/* capture phase: runs before children */}
```

- Every event bubbles except `onScroll`.
- `onChange` on inputs fires on every keystroke (like the native `input` event), not on blur.
- `onMouseEnter`/`onMouseLeave` don't bubble; `onMouseOver`/`onMouseOut` do.
- `onFocus`/`onBlur` do bubble in React (they map to `focusin`/`focusout`).

### Event types (TypeScript)

| Event | Type |
| --- | --- |
| click, mouse | `React.MouseEvent<HTMLButtonElement>` |
| input, select, textarea change | `React.ChangeEvent<HTMLInputElement>` |
| form submit | `React.SubmitEvent<HTMLFormElement>` (`FormEvent` is deprecated in @types/react 19.3) |
| key | `React.KeyboardEvent<HTMLInputElement>` |
| focus | `React.FocusEvent<HTMLInputElement>` |
| pointer | `React.PointerEvent<HTMLDivElement>` |
| drag | `React.DragEvent<HTMLDivElement>` |

To find a type, hover over `onChange` in your editor, or type the handler inline first and let TS infer it.

## Controlled inputs

The input's value lives in state; React is the single source of truth.

```tsx
function Signup() {
  const [email, setEmail] = useState('');
  const [agree, setAgree] = useState(false);
  const [plan, setPlan] = useState('free');

  return (
    <form>
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <input type="checkbox" checked={agree} onChange={(e) => setAgree(e.target.checked)} />
      <select value={plan} onChange={(e) => setPlan(e.target.value)}>
        <option value="free">Free</option>
        <option value="pro">Pro</option>
      </select>
      <textarea value={bio} onChange={(e) => setBio(e.target.value)} />  {/* value, not children */}
    </form>
  );
}
```

```tsx
// GOTCHAS
<input value={email} />                    // read-only: React warns (add onChange or readOnly)
<input value={user.name ?? ''} />           // never pass undefined/null; that switches controlled ↔ uncontrolled
<input type="number" value={n} onChange={(e) => setN(e.target.valueAsNumber)} /> // e.target.value is a string
// controlled input + async setState → caret jumps to the end. Always update synchronously.
```

```tsx
// many fields: one handler keyed by name
const [form, setForm] = useState({ first: '', last: '' });

function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
  setForm({ ...form, [e.target.name]: e.target.value });
}
<input name="first" value={form.first} onChange={handleChange} />
```

## Uncontrolled inputs

The DOM keeps the value. You read it on submit (with `FormData`) or through a ref. Less code, and no re-render on each keystroke.

```tsx
<input name="email" defaultValue="me@x.com" />     // initial value only
<input type="checkbox" name="agree" defaultChecked />
<input type="file" name="avatar" />                // file inputs are always uncontrolled

function handleSubmit(e: React.SubmitEvent<HTMLFormElement>) {
  e.preventDefault();
  const data = new FormData(e.currentTarget);
  const email = data.get('email') as string;
  const all = Object.fromEntries(data);            // { email: '...', agree: 'on' }
}
```

## Form actions (React 19)

`<form action={fn}>` takes a function. React calls it with the `FormData` inside a transition, and resets uncontrolled fields after it succeeds. This works with client functions and with Server Functions (`'use server'`), and with Server Functions the form works even before JavaScript loads (progressive enhancement).

```tsx
function Search() {
  async function search(formData: FormData) {
    const query = formData.get('q');
    await runSearch(String(query));
  }

  return (
    <form action={search}>
      <input name="q" />
      <button type="submit">Search</button>
      <button formAction={saveDraft}>Save draft</button>  {/* per-button action */}
    </form>
  );
}
```

To reset manually: `requestFormReset(form)` from `react-dom`. Since 19.3, the automatic reset fires the form's `onReset`.

### useActionState

Holds the result of the last action call, plus a pending flag. The action receives the previous state as its first argument.

```tsx
import { useActionState } from 'react';

type State = { error?: string; ok?: boolean };

async function subscribe(prev: State, formData: FormData): Promise<State> {
  const email = String(formData.get('email'));
  if (!email.includes('@')) return { error: 'Invalid email' };
  await api.subscribe(email);
  return { ok: true };
}

function Newsletter() {
  const [state, formAction, isPending] = useActionState(subscribe, {});
  // 3rd arg (permalink) is for Server Function forms: the URL to go to if submitted before hydration

  return (
    <form action={formAction}>
      <input name="email" defaultValue="" />
      <button disabled={isPending}>{isPending ? 'Sending…' : 'Subscribe'}</button>
      {state.error && <p role="alert">{state.error}</p>}
      {state.ok && <p>Thanks!</p>}
    </form>
  );
}
```

```tsx
// calling the dispatch outside a form: wrap it in a transition
const [, dispatch, isPending] = useActionState(addToCart, null);
<button onClick={() => startTransition(() => dispatch(productId))}>Add</button>
```

Calls are queued: each one waits for the previous one to finish and gets its result as `prev`.

### useFormStatus

Reads the pending state of the parent `<form>`, with no prop drilling. It must be called from a component rendered inside the form.

```tsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending, data, method, action } = useFormStatus();
  return <button type="submit" disabled={pending}>{pending ? 'Saving…' : 'Save'}</button>;
}

<form action={save}>
  <input name="title" />
  <SubmitButton />            {/* works */}
</form>
// useFormStatus() in the component that renders <form> → always pending: false
```

### useOptimistic

Shows a temporary value immediately while an async action runs. When the action finishes, the optimistic value is dropped and real state takes over. If the action fails, the UI falls back to the real state.

```tsx
import { useOptimistic, startTransition } from 'react';

function Thread({ messages, send }: { messages: Msg[]; send: (text: string) => Promise<void> }) {
  const [optimistic, addOptimistic] = useOptimistic(
    messages,
    (current: Msg[], text: string) => [...current, { text, sending: true }],
  );

  async function formAction(formData: FormData) {
    const text = String(formData.get('text'));
    addOptimistic(text);      // must run inside an action/transition (form actions already are)
    await send(text);         // parent updates `messages` when done
  }

  return (
    <>
      {optimistic.map((m, i) => <p key={i}>{m.text}{m.sending && ' (sending…)'}</p>)}
      <form action={formAction}><input name="text" /></form>
    </>
  );
}
```

## Form libraries and validation

Use built-in HTML validation first (`required`, `type="email"`, `minLength`, `pattern`). Always validate again on the server: client validation is UX, not security.

For large forms, React Hook Form (uncontrolled under the hood, few re-renders) plus a schema library like Zod is the common stack. TanStack Form is a type-safe alternative.

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.email(),                                   // Zod 4: top-level string formats
  age: z.coerce.number().int().min(18, 'Adults only'),
});
type FormValues = z.infer<typeof schema>;

function Profile() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } =
    useForm<FormValues>({ resolver: zodResolver(schema) });

  return (
    <form onSubmit={handleSubmit(async (values) => await save(values))}>
      <input {...register('email')} />                {/* name, ref, onChange, onBlur */}
      {errors.email && <span>{errors.email.message}</span>}
      <input type="number" {...register('age')} />
      <button disabled={isSubmitting}>Save</button>
    </form>
  );
}
```

The same Zod schema can validate the payload on the server (inside a Server Function or an API route).

<!-- nav -->
---

← [React: State](03-state.md) · [Index](README.md) · [React: Effects](05-effects.md) →
<!-- nav -->
