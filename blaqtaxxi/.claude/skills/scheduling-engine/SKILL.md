---
name: scheduling-engine
description: Use when working on lib/scheduler, lib/routing, lib/pricing, slot search, feasibility, rush-hour or traffic logic, late-risk, or any code that decides whether a ride can be booked. Enforces the pure-module contract and the test-first workflow.
---

# scheduling-engine

The correctness core of BLAQTAXXI. Read `docs/SCHEDULER_SPEC.md` fully before touching anything here.

## The rule that defines correct

A booking is valid only if
`prev.end + travel(prev.endPlace → new.pickup, departing prev.end) + buffer <= new.pickupAt`
**and**
`new.plannedEnd + travel(new.dropoff → next.startPlace, departing new.plannedEnd) + buffer <= next.startAt`,
inside the availability window, with no overlap. Equality passes.

## Workflow (always)

1. Find the matching case in `SCHEDULER_SPEC.md` §8 (S/L/V/P/C ids). If the change has no case, **write the case in the spec first**, then the test, then the code.
2. Write the failing test. Show it failing.
3. Implement the smallest change. Show it passing.
4. Run `npm run typecheck && npm test`.
5. Invoke the `scheduler-verifier` agent before reporting done.

## Purity contract (hard)

- `lib/scheduler/**` must not import `next`, `drizzle-orm`, `stripe`, `@upstash/*`, anything from `lib/db`, `lib/payments`, `lib/notify`, or use `fetch`, `process.env`, `Date.now()`, `new Date()` (no-arg), or `Math.random()`.
- Time is passed as `now`. Randomness lives only in test helpers with a seeded PRNG.
- Travel time enters only through `TravelTimeProvider`.
- Quick check: `grep -rnE "from '(next|drizzle-orm|stripe|@upstash)|process\.env|fetch\(|Date\.now\(|new Date\(\)|Math\.random" lib/scheduler` must return nothing.

## Time handling

- Store UTC, display `America/Chicago`. Use Luxon in `lib/time.ts` only; scheduler receives `Date`s.
- `dayWindow(localDate, tz)` builds the 05:00–24:00 window from local wall time, never a fixed UTC offset. Test on 2026-03-08 and 2026-11-01.
- Money is integer cents. Minutes are whole numbers, rounded **up**.

## Traffic model (demo provider)

- Departure-time lookup: the leg's departure time decides peak vs off-peak.
- Peaks 06:30–09:30 and 15:30–19:00 local. Factor ×1.4 when base < 15 min, else ×1.8, ceil.
- Must reproduce `docs/fixtures/travel-matrix.json → derivedExamples` exactly.
- Always **label demo travel times as illustrative** in UI and comments. Never present them as real traffic data.
- Known simplification: a leg that crosses the end of rush hour is priced by its start time. Say so in docs.

## Slot search

- Walk gaps, not a fixed hourly grid. First candidate = `prev.endAt + deadhead + buffer` snapped **up** to the 5-min grid.
- Enforce `maxProviderCallsPerSearch`; use the routing cache (cell × cell × hour bucket).
- Return suppressed times with a `reason` so the UI can explain "why is 9:00 gone?".

## Pricing

- Tiers from the off-peak **reference** duration: ≤30 → $20, >30–45 → $25, >45 → $30 (D-07). Price never depends on the hour. Cents only.

## Common mistakes to avoid

- Checking only the previous ride and forgetting the *next* pickup.
- Using the next ride's *pickup place* as the origin for the return leg but the wrong departure time (use `plannedEndAt`).
- Recomputing `plannedEndAt` for stored bookings; it is stored at booking time and stable.
- Using `<` where the spec says `<=` (S8b).
- Letting `TOO_SOON` apply to ride-now requests.
- Adding I/O "just for logging" inside the scheduler. Log at the caller.

## Definition of done

Spec case ids covered · tests fail-then-pass shown · purity grep clean · verifier agent clean · new assumptions added to `docs/DECISIONS.md`.
