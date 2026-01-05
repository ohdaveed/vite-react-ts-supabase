# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
bun install          # Install dependencies
bun run dev          # Start dev server
bun run build        # Type-check and build for production
bun run lint         # Run ESLint
bun run preview      # Preview production build
```

## Environment Setup

Create `.env` with:
```
VITE_SUPABASE_URL=<your-supabase-url>
VITE_SUPABASE_ANON_KEY=<your-supabase-anon-key>
```

For local Supabase: `npx supabase init && npx supabase start`, then apply `sample-db.sql` and enable Realtime on the `tasks` table.

## Path Alias

Use `@/*` for imports from `src/*` (configured in tsconfig.app.json and vite.config.ts).

## Architecture

**Provider Stack** (`src/App.tsx`): QueryClientProvider → AuthProvider → BrowserRouter → ToastContainer

**Auth Flow**:
- `AuthContext` (`src/context/AuthContext.tsx`) fetches current user on mount and subscribes to `onAuthStateChange`
- Access auth state via `useAuth` hook, not Supabase directly
- `ProtectedRoute` wraps authenticated routes, shows spinner while loading, redirects to `/login` if unauthenticated

**Data Layer**:
- All Supabase calls go through custom hooks in `src/hooks/`
- Queries use React Query with `queryKey: ["tasks"]`
- `useGetTasks` sets up realtime subscription on `tasks-changes` channel for INSERT/UPDATE/DELETE, updating cache via `queryClient.setQueryData`
- Mutations: `useAddTask`, `useDeleteTask`, `useSignOut`
- All task operations filter by `user.id` — maintain this pattern

**Realtime**: The `useGetTasks` hook manages Supabase channel subscriptions with proper cleanup. If adding fields to tasks, update the cache mapping functions in the realtime handlers.

**Routes**: `/` (HomePage), `/login` (LoginPage), `/dashboard` (ProtectedRoute → DashboardPage)

## Conventions

- Forms use `react-hook-form`
- Toasts via `react-toastify` — hooks throw errors for callers to toast
- Tailwind v4 via Vite plugin — styles in JSX classes only
- React Icons for icons
- New features: create hooks under `src/hooks/`, declare routes in `App.tsx`, protect with `ProtectedRoute` when auth-dependent
