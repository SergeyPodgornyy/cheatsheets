# React: Testing

## Stack

| Layer | Tool |
| --- | --- |
| Test runner | Vitest (Vite-native, Jest-compatible API); Jest for older setups |
| DOM environment | `jsdom` or `happy-dom`; or Vitest Browser Mode (real browser via Playwright) |
| Component testing | React Testing Library (`@testing-library/react`) |
| User interaction | `@testing-library/user-event` |
| Assertions on the DOM | `@testing-library/jest-dom` matchers |
| Network mocking | MSW (Mock Service Worker) |
| End-to-end | Playwright, Cypress |
| Visual / isolated components | Storybook (+ interaction and visual tests) |

```console
$ npm i -D vitest jsdom @testing-library/react @testing-library/user-event @testing-library/jest-dom
```

```ts
// vite.config.ts
export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/test/setup.ts',
  },
});
```

```ts
// src/test/setup.ts
import '@testing-library/jest-dom/vitest';
import { cleanup } from '@testing-library/react';
afterEach(() => cleanup());                    // unmount between tests (automatic with globals: true)
```

## Testing Library basics

The guiding principle is to test what the user sees and does, not implementation details (state, hooks, class names).

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Counter } from './Counter';

test('increments on click', async () => {
  const user = userEvent.setup();                 // set up before render
  render(<Counter initial={1} />);

  const button = screen.getByRole('button', { name: /increment/i });
  await user.click(button);                       // real event sequence: pointerdown, mousedown, focus, click...

  expect(screen.getByText('Count: 2')).toBeInTheDocument();
});
```

### Queries

| Prefix | 0 matches | 1 match | >1 | Async |
| --- | --- | --- | --- | --- |
| `getBy` | throw | return | throw | no |
| `queryBy` | `null` | return | throw | no |
| `findBy` | reject | resolve | reject | yes (retries, default 1000 ms) |
| `getAllBy` / `queryAllBy` / `findAllBy` | throw / `[]` / reject | array | array | `findAllBy` only |

Priority (most accessible first):

1. `getByRole('button', { name: 'Save' })`: the best default, and it checks accessibility as a side effect
2. `getByLabelText('Email')`: form fields
3. `getByPlaceholderText`, `getByText`, `getByDisplayValue`
4. `getByAltText`, `getByTitle`
5. `getByTestId('x')` (`data-testid`): last resort

```tsx
expect(screen.queryByText('Error')).not.toBeInTheDocument();  // assert absence with queryBy
expect(await screen.findByText('Loaded')).toBeVisible();       // wait for async UI

import { within, waitFor } from '@testing-library/react';
const row = screen.getByRole('row', { name: /alice/i });
within(row).getByRole('button', { name: 'Delete' });           // scoped query

await waitFor(() => expect(onSave).toHaveBeenCalledTimes(1));  // retry an assertion until it passes
```

### user-event

```tsx
const user = userEvent.setup();
await user.type(screen.getByLabelText('Name'), 'Ada');
await user.clear(input);
await user.selectOptions(screen.getByRole('combobox'), 'pro');
await user.keyboard('{Enter}');
await user.tab();
await user.upload(fileInput, new File(['x'], 'a.png', { type: 'image/png' }));
await user.hover(el);
```

Prefer `user-event` to `fireEvent`: it fires the whole realistic event chain and respects disabled elements and focus.

## Async, act, and timers

RTL wraps `render`, user-event, and `findBy`/`waitFor` in `act()` for you. A warning saying "An update to X inside a test was not wrapped in act(...)" means state changed after the test stopped waiting. Fix it by awaiting the resulting UI (`findBy`), not by adding `act` everywhere.

```tsx
import { act } from 'react';                       // React 19: import act from 'react', not test-utils

vi.useFakeTimers();
const user = userEvent.setup({ advanceTimers: vi.advanceTimersByTime });
render(<Toast />);
await act(() => vi.advanceTimersByTimeAsync(3000));
expect(screen.queryByRole('status')).not.toBeInTheDocument();
vi.useRealTimers();
```

## Mocking the network with MSW

MSW intercepts real `fetch` calls, so the code under test stays unchanged. The same handlers work in tests, Storybook, and local dev.

```ts
// src/test/server.ts
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';

export const server = setupServer(
  http.get('/api/user/:id', ({ params }) => HttpResponse.json({ id: params.id, name: 'Ada' })),
);

// setup.ts
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

```tsx
test('shows an error when the request fails', async () => {
  server.use(http.get('/api/user/:id', () => new HttpResponse(null, { status: 500 })));
  render(<Profile id="1" />, { wrapper: Providers });
  expect(await screen.findByRole('alert')).toHaveTextContent(/failed/i);
});
```

## Providers and custom render

```tsx
function renderWithProviders(ui: React.ReactElement) {
  const queryClient = new QueryClient({ defaultOptions: { queries: { retry: false } } }); // fresh per test
  return render(
    <QueryClientProvider client={queryClient}>
      <MemoryRouter initialEntries={['/users/1']}>{ui}</MemoryRouter>
    </QueryClientProvider>,
  );
}
```

## Testing hooks

```tsx
import { renderHook, act } from '@testing-library/react';

test('useCounter', () => {
  const { result, rerender } = renderHook(({ step }) => useCounter(step), {
    initialProps: { step: 1 },
  });
  act(() => result.current.increment());
  expect(result.current.count).toBe(1);
  rerender({ step: 5 });
});
```

Test hooks through a component that uses them when you can. `renderHook` is for reusable library-style hooks.

## Mocking modules

```ts
vi.mock('./analytics', () => ({ track: vi.fn() }));
import { track } from './analytics';
expect(track).toHaveBeenCalledWith('signup', expect.objectContaining({ plan: 'pro' }));

const onSubmit = vi.fn();
render(<Form onSubmit={onSubmit} />);
```

## Server Components and E2E

Async Server Components can't be rendered by RTL in jsdom yet. Test their data functions as plain async functions, test client components with RTL, and cover the full server–client flow with Playwright:

```ts
import { test, expect } from '@playwright/test';

test('signup flow', async ({ page }) => {
  await page.goto('/signup');
  await page.getByLabel('Email').fill('a@b.co');
  await page.getByRole('button', { name: 'Sign up' }).click();
  await expect(page.getByRole('heading', { name: 'Welcome' })).toBeVisible();
});
```

Accessibility checks: `vitest-axe`/`jest-axe` in unit tests, `@axe-core/playwright` in E2E.

<!-- nav -->
---

← [React: TypeScript](15-typescript.md) · [Index](README.md) · [React: Patterns, Accessibility & Security](17-patterns.md) →
<!-- nav -->
