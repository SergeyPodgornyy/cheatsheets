# React: Data Fetching & State Libraries

## Server state vs client state

- **Server state** lives on a backend: users, posts, orders. It's async, shared, and can go stale. You need caching, deduplication, refetching, pagination, and invalidation. Tools: TanStack Query, SWR, RTK Query, Apollo/Relay (GraphQL), framework loaders, RSC.
- **Client state** exists only in the browser: UI toggles, form drafts, selections, wizard steps. Use `useState`/`useReducer`, context, or a store (Zustand, Redux Toolkit, Jotai).

Most "global state" turns out to be server state. Once a query library handles that, the client store is usually small.

## TanStack Query (v5)

```console
$ npm i @tanstack/react-query @tanstack/react-query-devtools
```

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

const queryClient = new QueryClient({
  defaultOptions: { queries: { staleTime: 60_000, retry: 2 } },
});

<QueryClientProvider client={queryClient}>
  <App />
  <ReactQueryDevtools />
</QueryClientProvider>
```

### Queries

```tsx
import { useQuery, queryOptions, keepPreviousData } from '@tanstack/react-query';

// queryOptions: one typed definition reused by useQuery, prefetch, and invalidation
export const todoQuery = (id: number) => queryOptions({
  queryKey: ['todos', id],                         // cache key: arrays, hierarchical
  queryFn: async ({ signal }) => {
    const res = await fetch(`/api/todos/${id}`, { signal });   // auto-cancel when unused
    if (!res.ok) throw new Error(`HTTP ${res.status}`);        // fetch doesn't throw on 4xx/5xx
    return res.json() as Promise<Todo>;
  },
});

function TodoView({ id }: { id: number }) {
  const { data, isPending, isError, error, isFetching, refetch } = useQuery(todoQuery(id));

  if (isPending) return <Spinner />;               // no data yet (v5: isLoading = isPending && isFetching)
  if (isError) return <p>{error.message}</p>;
  return <h1>{data.title}{isFetching && ' ↻'}</h1>;   // isFetching: background refresh
}
```

```tsx
// common options
useQuery({
  queryKey: ['todos', { page, filter }],           // every variable the fn uses belongs in the key
  queryFn: () => fetchTodos(page, filter),
  enabled: !!userId,                               // dependent query: wait for the input
  placeholderData: keepPreviousData,               // pagination: keep the old page while the next loads
  select: (todos) => todos.filter((t) => !t.done), // derive or transform; re-renders only if the result changes
  refetchInterval: 10_000,                         // polling
  staleTime: Infinity,                             // never refetch automatically
});
```

| Option | Default | Meaning |
| --- | --- | --- |
| `staleTime` | `0` | how long data counts as fresh (no refetch while fresh) |
| `gcTime` | 5 min | how long unused cache entries are kept |
| `refetchOnWindowFocus` | `true` | refetch stale data when the tab regains focus |
| `retry` | 3 (client) | retries with exponential backoff |

### Mutations

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function AddTodo() {
  const qc = useQueryClient();
  const mutation = useMutation({
    mutationFn: (text: string) => api.post('/todos', { text }),
    onSuccess: () => qc.invalidateQueries({ queryKey: ['todos'] }), // refetch everything under ['todos', ...]
  });

  return (
    <button disabled={mutation.isPending} onClick={() => mutation.mutate('Buy milk')}>
      {mutation.isPending ? 'Adding…' : 'Add'}
    </button>
  );
}
```

```tsx
// optimistic update with rollback
useMutation({
  mutationFn: updateTodo,
  onMutate: async (next, context) => {
    await context.client.cancelQueries({ queryKey: ['todos', next.id] });
    const previous = context.client.getQueryData(['todos', next.id]);
    context.client.setQueryData(['todos', next.id], next);
    return { previous };
  },
  onError: (_err, next, result, context) => context.client.setQueryData(['todos', next.id], result?.previous),
  onSettled: (_d, _e, next, _r, context) => context.client.invalidateQueries({ queryKey: ['todos', next.id] }),
});
// simpler option: render mutation.variables as a pending item while mutation.isPending
```

### Suspense, infinite lists, prefetching

```tsx
const { data } = useSuspenseQuery(todoQuery(id));   // data is never undefined; wrap in <Suspense> + error boundary

const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
  queryKey: ['feed'],
  queryFn: ({ pageParam }) => fetchFeed(pageParam),
  initialPageParam: 0,
  getNextPageParam: (last) => last.nextCursor ?? undefined,  // undefined → no more pages
});
data?.pages.flatMap((p) => p.items);

// prefetch on hover, or in a router loader
queryClient.prefetchQuery(todoQuery(id));
await queryClient.ensureQueryData(todoQuery(id));   // in a loader: returns cached data or fetches it
```

With SSR or RSC, prefetch on the server into a `QueryClient`, then pass the cache to the client with `dehydrate` and `<HydrationBoundary state={...}>`.

## SWR

A lighter alternative from Vercel: `const { data, error, isLoading, mutate } = useSWR('/api/user', fetcher)`. Stale-while-revalidate caching, deduplication, refetch on focus.

## Zustand (v5)

A minimal store without providers. Components subscribe with selectors and re-render only when the selected slice changes.

```tsx
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';
import { useShallow } from 'zustand/react/shallow';

type CartState = {
  items: Item[];
  add: (item: Item) => void;
  remove: (id: string) => void;
  clear: () => void;
};

export const useCart = create<CartState>()(           // curried create<T>()(...) for TS inference
  devtools(
    persist(
      (set) => ({
        items: [],
        add: (item) => set((s) => ({ items: [...s.items, item] })),   // set merges shallowly
        remove: (id) => set((s) => ({ items: s.items.filter((i) => i.id !== id) })),
        clear: () => set({ items: [] }),
      }),
      { name: 'cart' },                                // localStorage key
    ),
  ),
);

function CartCount() {
  const count = useCart((s) => s.items.length);       // re-renders only when the length changes
  return <span>{count}</span>;
}

function CartActions() {
  const { add, clear } = useCart(useShallow((s) => ({ add: s.add, clear: s.clear })));
  // GOTCHA (v5): a selector returning a new object/array without useShallow → infinite loop
}

useCart.getState().clear();                           // outside React (tests, event listeners)
useCart.subscribe((s) => console.log(s.items));
```

## Redux Toolkit

The official, modern way to write Redux: slices with Immer built in, typed hooks, and RTK Query for data fetching. Worth it for large teams that want strict conventions, middleware, and time-travel DevTools.

```tsx
import { configureStore, createSlice, type PayloadAction } from '@reduxjs/toolkit';
import { Provider, useDispatch, useSelector } from 'react-redux';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => { state.value += 1; },              // Immer: "mutations" are safe here
    addBy: (state, action: PayloadAction<number>) => { state.value += action.payload; },
  },
});

export const { increment, addBy } = counterSlice.actions;
export const store = configureStore({ reducer: { counter: counterSlice.reducer } });

type RootState = ReturnType<typeof store.getState>;
type AppDispatch = typeof store.dispatch;
export const useAppSelector = useSelector.withTypes<RootState>();
export const useAppDispatch = useDispatch.withTypes<AppDispatch>();

<Provider store={store}><App /></Provider>

const value = useAppSelector((s) => s.counter.value);
const dispatch = useAppDispatch();
dispatch(addBy(5));
```

## Jotai and others

- Jotai: atomic state, bottom-up. `const countAtom = atom(0)` → `const [count, setCount] = useAtom(countAtom)`. Derived atoms: `atom((get) => get(countAtom) * 2)`. Re-renders are fine-grained, and no selectors are needed.
- Valtio: proxy-based, mutable-looking state (`state.count++`) with `useSnapshot`.
- XState / XState Store: state machines for complex flows (checkout, wizards, media players).
- TanStack Store: framework-agnostic store used inside the TanStack libraries.

| Library | Model | When |
| --- | --- | --- |
| Zustand | one store + selectors | default pick for client state |
| Redux Toolkit | slices, actions, middleware | big apps, strict patterns, existing Redux |
| Jotai | atoms | many independent bits of state, derived values |
| XState | state machines | complex, explicit state transitions |

## HTTP clients

`fetch` is built in, and it does not reject on HTTP errors: check `res.ok`. Wrappers: `ky` (small, retries, hooks) or `axios` (interceptors, older API). For end-to-end types: tRPC, OpenAPI codegen (`openapi-typescript` + `openapi-fetch`), or GraphQL codegen.

<!-- nav -->
---

← [React: Routing](13-routing.md) · [Index](README.md) · [React: TypeScript](15-typescript.md) →
<!-- nav -->
