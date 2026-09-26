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
- On `demo` the UI shows a `StatusChip` when running on a demo implementation ("Demo travel times", "Test payment (simulated)", "SMS not connected"). Chips derive from the same env check, so they can't lie.
- **Chips render only when `NEXT_PUBLIC_SHOW_TODO_CHIPS=true`.** A production build renders none, and CI fails if a chip is tied to a launch blocker in `docs/UNKNOWNS.md`. Customers never see unfinished work.
- **`APP_ENV` = `demo | pilot | production`.** Demo fallbacks run only in `demo`. `pilot` (IAS-hosted, Stripe **test**, public may use it up to payment) and `production` (the client's own deployment) **fail loudly at boot** if any required integration is missing. The dev sign-in and `/dev/*` routes must not exist in either. `pilot` adds a `PilotBanner`, a "Send feedback" control, and the payment-step notice, and never shows unfinished-work chips to the public.

## Matrix

See `docs/ARCHITECTURE.md` §9 for the full env → fallback table. Keep it current; if you add an integration, add a row.

## Chips

- `TodoChip`: something planned but not built (e.g., "Cash App/Zelle", "Airport logic", "Brand logo"). Visible in `demo` only, links to the relevant doc or decision id. Style with neutral `blaq-*` tokens.
- `StatusChip`: current runtime mode of a subsystem (demo/live/test).
- In `demo`, incomplete is never hidden: visible gaps are integrity signals and part of the demonstration. **In `pilot` and `production` nothing unfinished may be visible to the public**; known gaps go on the pilot's "Known limitations" page (generated from `docs/UNKNOWNS.md`) and unfinished launch blockers stay tracked there.

## Rules

- No secrets in the repo. `.env.example` lists names and comments only. Real values go in `.env.local`.
- Stripe is **test mode** in `demo` and `pilot`. A live key loads only in the client's own production deployment after the Phase 10 sign-off. IAS's Stripe is always test mode and never processes his fares (D-103, D-119).
- Third-party calls must be rate-limited and cached where they cost money (routing especially).
- No dependency on a vendor's SDK in `lib/scheduler`.

## Never

- `output: 'export'` in `next.config` (silently kills API routes).
- `next/font/google` (fails when fonts.googleapis.com is blocked); use a runtime `<link>`.
- Cherry-picking files from different patch drops; apply full consolidated patches.

## Definition of done for a new seam

Interface + real + demo implementations · factory reads env in the function body · StatusChip present · matrix row added · test proving demo mode works with no env · `.env.example` updated.
