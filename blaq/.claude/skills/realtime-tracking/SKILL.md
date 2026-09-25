---
name: realtime-tracking
description: Use when building the driver location drop, driver PWA pings, the rider ride page, ETA calculation, the T-30 visibility gate, staleness handling, or the driver-simulator dev tool.
---

# realtime-tracking

Read `docs/ARCHITECTURE.md` §2, §7 and `docs/SCHEDULER_SPEC.md` §6 first.

## Non-negotiables

1. **n8n is never in this path.** Ping → Redis → ETA → rider screen, all inside the Next.js route handlers.
2. **T-30 gate is server-enforced.** The API refuses location outside the window (`403 not_yet_visible`). Hiding it in the UI is not a gate. Use the pure `visibilityWindow(pickupAt, now, status)` from `lib/scheduler`.
3. **Honest about freshness.** iOS PWAs cannot track in the background. Show "last seen N s ago", flag stale pings (>90 s), never render a stale dot as live.
4. **Privacy by construction.** A rider sees only their own booking and, inside their window, the driver's position. No other rider's data ever crosses the API. Pings are transient (Redis TTL 120 s; optional trail ≤ 24 h), never written to Postgres.

## Driver client (PWA at `/drive`)

- `navigator.geolocation.watchPosition` while the console is open, throttled to one POST per 15–30 s (config in `lib/config/tracking.ts`). Also send a ping on every status tap.
- Request permission with a plain-language reason. Handle denied/unavailable states with a visible chip, not a silent failure.
- Keep the screen awake while a ride is active where the platform allows (Wake Lock API, feature-detected); treat it as best-effort.
- 44 px minimum touch targets. Status buttons: Heading to pickup → Arrived → Rider in car → Complete (+ No-show).

## Server

- `POST /api/driver/ping`: Zod-validate `{lat,lng,accuracy,at}`, clamp absurd values, write `driver:{id}:loc` (TTL 120 s), then run `projectDay` (cached per ping timestamp) and emit a late-risk event if the risk state changed. Emit **only on change** to avoid spamming n8n.
- `GET /api/rides/[token]`: verify token hash → compute `visibilityWindow` → `countdown` returns only the countdown; `live` returns `{driver:{lat,lng,seenSecondsAgo,stale}, etaMin, pickup}`; `done` returns receipt.
- ETA = `provider.minutes(pingPlace, pickupPlace, now)` through the routing cache; memoize per `(bookingId, pingAt)`.

## Rider client (`/ride/[token]`)

- Poll every 60 s before T-30, every 10 s inside the window. Stop polling when done. Back off on errors.
- Countdown uses Space Mono with tabular figures; announce meaningful changes via `aria-live="polite"` (not every tick).
- Only one Emerald use in the view: the "Live" badge.

## Dev simulator

`/dev/sim` (disabled in production) replays a route from the fixtures at 10× speed by POSTing pings, so the rider screen is demoable at a desk with no phone.

## Tests to write first

V1 (visibility), L1/L2 (late-risk + cascade), API test proving no location leaks at T-31 min and is returned at T-29, stale-ping flag, token expiry, rate limiting.

## Pitfalls

- Reading `NEXT_PUBLIC_*` through a module-level const chain → `undefined` in prod. Read inside the function.
- Comparing local times: convert with `America/Chicago` once, in `lib/time.ts`.
- Clock skew: trust server time for windows; only use the device timestamp for "seen N s ago" display, clamped to ≥ 0.
- Do not build WebSocket/SSE for the MVP (gotcha #7).
