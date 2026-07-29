# Convention: Application Architecture

> Scope: Next.js frontend (App Router). The Fastify backend has its own separate convention (see dedicated issue).
> Reference methodology: [Feature-Sliced Design](https://feature-sliced.design/docs/get-started/overview) (FSD), adapted to avoid conflicts with Next.js routing.

## 1. Why FSD

FSD splits code into **layers**, then **slices** (business domains), then **segments** (technical purpose). The goal: a module can only import from layers strictly below it. This avoids circular dependencies and the kind of entanglement that makes a project unreadable over time, and, by extension, makes it easier for an LLM to read and modify without breaking things elsewhere.

## 2. Folder structure

```
app/                     ← Next.js routing, at the root, thin wrappers only
  (dashboard)/
    page.tsx
  api/
    ...   ← route handlers, case by case (see §5)

src/
  _app/                  ← former FSD "app" layer (renamed to avoid collision with Next's app/)
    providers/
    styles/
    config/

  _pages/                ← former FSD "pages" layer (renamed to avoid collision with Next's pages/)
    dashboard/
      ui/
      index.ts           ← slice's Public API

  widgets/
    header/
      ui/
      index.ts

  features/
    login/
      ui/
      model/
      api/
      index.ts

  entities/
    user/
      model/
      api/
      index.ts

  shared/
    ui/
    api/
    lib/
    config/
```

**Why `app/` (root) vs `src/`:** Next.js requires `app/` to define routes. We keep that folder for routing only, and all FSD code lives in `src/`. The FSD `app` and `pages` layers are renamed `_app` and `_pages` (underscore prefix) to avoid a naming collision with Next's own folders, this is the convention recommended by the official FSD docs, and it's compatible with their `steiger` linter.

## 3. Layers, top to bottom

| Layer | Role |
|---|---|
| `_app` | Global providers, global styles, routing config, app-wide setup |
| `_pages` | Composes a page from widgets/features/entities |
| `widgets` | Large self-contained UI blocks (header, sidebar, full feed) |
| `features` | Reusable business-value actions (login, add to cart, like) |
| `entities` | Business entities (user, product, order) |
| `shared` | Generic code, no business logic (UI kit, helpers, generic API client) |

**Import rule**: a layer can only import from layers **strictly below** it.
`_pages → widgets → features → entities → shared`
A layer can never import from a layer above it, nor from another slice on the same layer (e.g. the `login` feature must not directly import the `signup` feature).

## 4. Slices and segments

- A **slice** = a split by business domain within a layer (`features/login`, `entities/user`...).
- A slice cannot import another slice on the same layer.
- A **segment** = a split by technical purpose within a slice:
  - `ui`, display components
  - `api`, backend calls, request/response types, mappers
  - `model`, state, schemas, business logic
  - `lib`, utilities internal to the slice
  - `config`, configuration, feature flags

## 5. Public API per slice

Each slice exposes an `index.ts` file that explicitly lists what's public. Everything else in the slice is considered private: **never import a slice's internal file directly**, only through its `index.ts`.

```ts
// src/features/login/index.ts
export { LoginForm } from './ui/login-form';
export { useLogin } from './model/use-login';
```

```ts
// ✅ correct
import { LoginForm } from '@/features/login';

// ❌ not allowed, direct access to the slice's internals
import { LoginForm } from '@/features/login/ui/login-form';
```

No tooled enforcement for now (no `steiger`-style linter), this is a manual discipline for now. Worth revisiting if the number of slices grows and direct internal imports start slipping in.

## 6. Next.js-specific cases

- **`app/page.tsx` is a thin wrapper**: it imports from `_pages`, `widgets`, or `features`, and contains no state, fetching, or business logic.
- **Route handlers (`app/api/.../route.ts`)**: to be handled case by case. Since the Fastify backend has its own convention, the question to settle project by project is: do these routes stay as lightweight Next endpoints (proxy, webhook, edge-specific), or do they migrate to Fastify as soon as there's real business logic? No fixed general rule here for now.
- **Server vs Client Components**: a `ui` component can be Server or Client depending on the need; this doesn't change which layer/slice it belongs to, only its file (`'use client'` at the top if needed).

## 7. Concrete example

Complete "login" feature:

```
src/features/login/
  ui/
    login-form.tsx
  model/
    use-login.ts
  api/
    login-request.ts
  index.ts
```

```ts
// api/login-request.ts
export async function loginRequest(email: string, password: string) {
  const res = await fetch('/api/auth/login', {
    method: 'POST',
    body: JSON.stringify({ email, password }),
  });
  if (!res.ok) throw new Error('Login failed');
  return res.json();
}
```

```ts
// model/use-login.ts
import { useState } from 'react';
import { loginRequest } from '../api/login-request';

export function useLogin() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function login(email: string, password: string) {
    setLoading(true);
    setError(null);
    try {
      await loginRequest(email, password);
    } catch (e) {
      setError((e as Error).message);
    } finally {
      setLoading(false);
    }
  }

  return { login, loading, error };
}
```

```ts
// index.ts
export { LoginForm } from './ui/login-form';
export { useLogin } from './model/use-login';
```

## 8. Open points / to revisit

- What happens to Next.js route handlers (`app/api`) relative to the Fastify backend.
- Moving to tooled enforcement (`steiger`) if the project grows.
