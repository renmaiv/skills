---
name: nextjs-app-refactor
description: >
  Splits mixed Server/Client Next.js pages into clean boundaries — extracts client logic into `*Client.tsx`,
  rewrites `page.tsx` as a pure Server Component, flags repeated JSX for componentisation, and detects
  styling inconsistencies across the codebase. Trigger on: refactor requests, mixed Server/Client files,
  pages >200 LOC with hooks, duplicated JSX, or any style drift between components.
---

## Always: Confirm Before Acting

State: what was detected, why it's a problem, what you'll change. **Wait for approval before modifying any file.**

---

## Core Rules

- `metadata` never in a `"use client"` file — use `layout.tsx` if it spans multiple routes
- Hooks (`useState`, `useEffect`, `useRouter`, etc.) must live in Client Components
- No partial refactors, duplicate component definitions, or JSX outside component scope
- Same JSX structure 2+ times inline → extract into a named component

---

## When to Refactor

- `metadata` in a Client Component, or Server/Client concerns mixed in one file
- Page >200–300 LOC with hooks
- User says "refactor"
- Style drift detected (see below)

---

## Refactor Steps

1. **Create** `app/<route>/<FeatureName>Client.tsx` — move `"use client"`, all hooks, JSX, handlers
2. **Rewrite** `page.tsx`:
```tsx
import type { Metadata } from 'next';
import FeatureNameClient from './FeatureNameClient';
export const metadata: Metadata = { /* ... */ };
export default function Page() { return <FeatureNameClient />; }
```
3. **Validate** — no `"use client"` or hooks in `page.tsx`, no syntax errors, project builds

---

## Repeated Structure Detection

Flag 2+ JSX blocks with identical structure (same wrapper, classNames, child layout) differing only in data values, or `.map()` calls rendering >3 elements per item. Extract into a domain-named component (`ProjectSection`, `WorkEntry`) — not `Section1`. Confirm before extracting.

---

## Styling Consistency Detection

During any refactor or property change, scan for drift. Flag when **2+ places** share an apparently intentional value but express it inconsistently.

| Category | What to flag |
|---|---|
| **Layout & Spacing** | Duplicate/overriding utilities on one element (`px-4 px-6`); inconsistent scale values across instances; same pattern using different responsive prefixes (`md:` vs `lg:`); `style={{}}` overrides conflicting with classNames; same component with diverging Tailwind variants across call sites |
| **Color & Tokens** | `text-[#3B82F6]` vs `text-blue-500` — hardcoded hex matching an existing token; `shadow-md`, `shadow-lg`, inline `box-shadow` mixed for the same elevation level |
| **Structure & Scale** | `<Button />` exists but some sites recreate it with raw classes; same class defined differently across `.module.css` files; hardcoded z-index values with no defined scale |

When detected: list affected files/locations, propose a CSS variable/Tailwind token or base component, wait for approval.

---

## Output

Full file contents only. No placeholders. Code must compile.
