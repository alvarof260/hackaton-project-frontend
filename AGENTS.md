# Repository Guidelines

## Project Overview

**CommiFlow** — Plataforma de comisiones artísticas. El frontend es una SPA en React 19 que consume una API .NET 10.

| Component | Location | Tech Stack |
| :--- | :--- | :--- |
| Frontend (SPA) | `/src` | React 19, TypeScript 5.9, Vite 8 |
| UI Components | `src/components` | HeroUI v3, Tailwind CSS v4 |
| Routing | `src/pages` | React Router v6 |
| State | `src/stores` | Zustand |
| Forms | `src/features/*/components` | React Hook Form + Zod 4 |
| API Client | `src/api` | fetch (typed) → .NET 10 backend |

## Folder Structure

```
src/
├── api/              # Typed API client (fetch wrappers)
├── components/       # Shared/presentational components (Button, Card, etc.)
├── features/         # Domain-driven modules
│   ├── auth/         # Login, Register, JWT handling
│   │   └── components/
│   ├── artist/       # Public artist wall (profile, gallery, tiers)
│   │   └── components/
│   ├── commission/   # Calculator, commission form, checkout
│   │   └── components/
│   └── dashboard/    # Artist backoffice (orders, status management)
│       └── components/
├── pages/            # Route-level components (one per route)
├── stores/           # Zustand stores (calculator, auth, etc.)
├── types/            # Shared TypeScript types (API responses, domain models)
├── utils/            # Pure helper functions
├── index.css         # Global styles (tailwind + heroui imports)
└── main.tsx          # Entry point + router setup
```

## Available Skills

### Project-Specific Skills

| Skill | Description | Trigger |
| :--- | :--- | :--- |
| `commis-commit` | Commit conventions (Conventional Commits 1.0.0) | Creating, reviewing, or amending commits |

### Generic Skills

| Skill | Description | Trigger |
| :--- | :--- | :--- |
| `heroui` | HeroUI v3 components, styling, theming, compound patterns | Importing/creating HeroUI components, styling |
| `typescript` | Strict TS patterns, const types, utility types, `import type` | Writing types, interfaces, generics |
| `tailwind-4` | Tailwind v4 patterns: `cn()`, theme vars, no `var()` in className | Styling components with Tailwind |
| `react-19` | React 19 + React Compiler: no `useMemo`/`useCallback`, `ref` as prop | Writing components or hooks in `.tsx` |
| `react-router` | React Router v6: routes, navigation, guards, params | Setting up routes, `useNavigate`, `useParams` |
| `react-hook-form` | Form handling, validation, controlled/uncontrolled inputs | Creating or modifying forms |
| `zod-4` | Schema validation, breaking changes from v3, `zodResolver` | Writing validation schemas |

## Auto-invoke Skills

| Action | Skill |
| :--- | :--- |
| Creating, reviewing, or amending commits | `commis-commit` |
| Creating or styling a HeroUI component | `heroui` |
| Writing TypeScript types or interfaces | `typescript` |
| Writing CSS / Tailwind classes | `tailwind-4` |
| Writing React components (`.tsx`) | `react-19` |
| Setting up routes or navigation | `react-router` |
| Creating or modifying forms | `react-hook-form` |
| Writing validation schemas | `zod-4` |

## Key Domain Concepts

| Term | Meaning |
| :--- | :--- |
| **Tier** | Service level offered by artist (e.g. "Boceto", "Full Color") |
| **Addon** | Price modifier on a tier (fixed amount or percentage multiplier) |
| **Commission** | A client order: tier + addons + brief + references |
| **SnapshotData** | Immutable JSON price breakdown stored at purchase time |
| **IsQueueOpen** | Artist master toggle — blocks new commissions when `false` |

## State Machine (Commission Status)

```
PendingApproval → AwaitingPayment → InQueue → WIP → Completed
       ↓
  Cancelled (artist rejects)
```

## Commands

```bash
bun dev        # Start Vite dev server
bun build      # TypeScript check + Vite production build
bun lint       # ESLint
```
