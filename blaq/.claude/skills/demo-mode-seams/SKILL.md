---
name: demo-mode-seams
description: Use when adding or changing any external integration (routing, Places, Stripe, Postgres, Redis, email, SMS, n8n, auth) so it has an env-var check with a graceful demo fallback, and when adding TodoChip/StatusChip markers for incomplete work.
---

# demo-mode-seams

Goal: `npm run dev` with **zero credentials** gives a fully clickable app. Gaps are visible, never hidden.

## The seam pattern

One module per integration, one factory, one interface:

```ts
// lib/<thing>/index.ts
export interface Thing { /* the minimum the app needs */ }
export function getThing(): Thing {
  // Read env INSIDE the function (NEXT_PUBLIC_* are inlined at build time; no module-level const chains).
  const key = process.env.THING_KEY;
  return key ? realThing(key) : demoThing();
}
```

- The app depends on the interface, never on the vendor SDK.
- The demo implementation is deterministic and offline (fixtures, in-memory Map with TTL, console/outbox logging).
- The UI shows a `StatusChip` when running on a demo implementation ("Demo travel times", "Test payment (simulated)", "SMS not connected"). Chips derive from the same env check, so they can't lie.
- Production must fail loudly if a required integration is missing (throw at boot in `NODE_ENV=production`), while preview/dev falls back.

## Matrix

See `docs/ARCHITECTURE.md` §8 for the full env → fallback table. Keep it current; if you add an integration, add a row.

## Chips

- `TodoChip`: something planned but not built (e.g., "Cash App/Zelle", "Airport logic", "Brand logo"). Visible on-page, links to the relevant doc or decision id.
- `StatusChip`: current runtime mode of a subsystem (demo/live/test).
- Incomplete is never hidden. Visible gaps are integrity signals, and they are part of the demonstration ("demonstrate, never claim").

## Rules

- No secrets in the repo. `.env.example` lists names and comments only. Real values go in `.env.local`.
- Stripe is **test mode** unless Steve says otherwise in writing. The live key must never load in preview.
- Third-party calls must be rate-limited and cached where they cost money (routing especially).
- No dependency on a vendor's SDK in `lib/scheduler`.

## Never

- `output: 'export'` in `next.config` (silently kills API routes).
- `next/font/google` (fails when fonts.googleapis.com is blocked); use a runtime `<link>`.
- Cherry-picking files from different patch drops; apply full consolidated patches.

## Definition of done for a new seam

Interface + real + demo implementations · factory reads env in the function body · StatusChip present · matrix row added · test proving demo mode works with no env · `.env.example` updated.
