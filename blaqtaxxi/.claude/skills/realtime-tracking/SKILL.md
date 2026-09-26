---
name: realtime-tracking
description: Use when building the driver console navigation flow, the driver location drop, the customer trip page at /pickup/[link] (before, approaching, during, after), ETA, the T-30 visibility gate, staleness, the Google Maps deep-link builder, the customer link scheme, or the driver simulator.
---

# realtime-tracking

Read `docs/ARCHITECTURE.md` §2, §5, §8, `docs/SCREEN_MAP.md` §A–B, `docs/SCHEDULER_SPEC.md` §6, and `docs/UNKNOWNS.md` (U-D1, U-L1) first.

## Non-negotiables

1. **n8n is never in this path.** Ping → Redis → ETA → customer page, all inside Next.js route handlers.
2. **The T-30 gate is server-enforced.** The API refuses location outside the window (`403 not_yet_visible`). Hiding it in the UI is not a gate. Use the pure `visibilityWindow(pickupAt, now, status)` from `lib/scheduler`.
3. **Honest about freshness.** Show "last seen N s ago", flag pings older than 90 s, never render a stale marker as live.
4. **Privacy by construction.** A customer sees only their own booking and, inside their window, the driver's position. No other customer's data ever crosses the API. Pings are transient (Redis TTL 120 s), never in Postgres.
5. **Do not build on the background-location assumption.** See "The open risk" below. If U-D1 is unresolved when you reach Phase 5, stop and ask.

## The open risk (U-D1)

- **Verified:** Google's Navigation SDK is Android/iOS (plus Flutter/React Native) only, no web version. Google Maps URLs need no API key; `dir_action=navigate` starts turn-by-turn on mobile.
- **Unverified (spike SP-1, on the client's phone):** a backgrounded PWA stops reporting location, so the customer's map would freeze while the driver is in Google Maps.
- **Decision tree:** if the spike shows continuous pings → foreground `watchPosition` + ping on each status tap. If not → propose a thin native shell for `/drive` only (background location plugin, still loads the Next.js UI, still deep-links to Google Maps). That changes distribution and the stack footprint, so **ask before building it**.

## Driver console (`/drive`)

States and their one primary action (44 px buttons):

| State | Shows | Primary action |
|---|---|---|
| Idle / between rides | Next pickup, time to spare, late-risk | **Navigate** to pickup |
| Heading to pickup | ETA | **Arrived** |
| At pickup | Wait timer (5 min), call/text | **Rider in car** (and **No-show** once the wait elapses) |
| In trip | Route ETA | **Navigate to drop-off**, then **Complete** |

- Every button tap sends a location ping and a status transition (`POST /api/driver/status`).
- **`lib/navigation`** builds deep links; unit-test it (encoding, precision, missing coordinates):
  `https://www.google.com/maps/dir/?api=1&destination=<lat>,<lng>&travelmode=driving&dir_action=navigate`
  Use stored coordinates, not free-text addresses. Google Maps is the default; other apps are a later setting (U-D2).
- Ask the customer's last name at pickup to confirm identity (D-92). Never display the full link secret to the driver.
- Request geolocation with a plain-language reason; handle denied/unavailable with a visible state (SCREEN_MAP D8).

## Server

- `POST /api/driver/ping`: Zod-validate `{lat,lng,accuracy,at}`, clamp absurd values, write `driver:{id}:loc` (TTL 120 s), run `projectDay` (cached per ping timestamp), emit a late-risk event **only on change**.
- `GET /api/pickup/[link]`: verify the link (§ below) → compute `visibilityWindow` → return the phase payload: `before` (details, countdown), `approaching` (driver position with `seenSecondsAgo`/`stale`, ETA to pickup), `in_trip` (car position, route, ETA to drop-off), `after` (log, receipt). No cancel in trip.
- ETA via `provider.minutes(...)` through the routing cache; memoize per `(bookingId, pingAt)`. Fetch the route polyline once at trip start and store it on the booking.

## Customer trip page (`/pickup/[link]`)

- Load `layout-system` and start from `active-trip` / `fare-summary` (see `SCREEN_MAP.md` C6–C9). No bottom nav, no account UI.
- Map via a `MapView` component (Google Maps JS, browser key); on `demo` a labeled static map. Always put a **text ETA** next to the map for accessibility.
- Poll every 60 s before T-30, every 10 s inside the window; stop when done; back off on errors.
- Announce meaningful changes with `aria-live="polite"`, not every tick. Times and prices use tabular figures.
- Copy: "Check back 30 minutes before your trip to see where your driver is." / "Driver is 12 min away."
- Late-risk uses navy text + icon + label; **not red** (D-73).

## Customer link (`lib/links`) — treat as a bearer secret

- Recommended format `/pickup/<lastname>-<last4>-<random>` (U-L1). **Never authorize on last name + last 4 alone.**
- Random part: at least 64 bits from a CSPRNG; store only a hash; constant-time compare; expire 90 days after the trip; per-IP and per-prefix rate limits with lockout; strict limits on recovery.
- Normalize surnames (O'Brien → obrien, De La Cruz → delacruz, accents, hyphens); store the display name separately.
- Scrub `/pickup/*` from logs and analytics; page sends `Referrer-Policy: no-referrer` and `noindex`.
- **Tests first:** entropy, collisions (two customers, same surname and last 4), normalization, expiry, brute-force lockout, recovery flow, cross-customer isolation.

## Dev simulator

`/dev/sim` (never in production) replays a route from fixtures at 10× speed by POSTing pings so the customer and driver screens are demoable at a desk.

## Tests to write first

V1 (visibility), L1/L2 (late-risk + cascade), API test that no location leaks at T-31 min and is returned at T-29, no cross-customer leakage, stale-ping flag, deep-link builder, link security suite, rate limiting.

## Pitfalls

- `NEXT_PUBLIC_*` read through a module-level const → `undefined` in production. Read inside the function.
- Local times: convert with `America/Chicago` once, in `lib/time.ts`.
- Clock skew: trust server time for windows; use the device timestamp only for "seen N s ago", clamped to ≥ 0.
- No WebSocket/SSE for the MVP (gotcha 7).
- Do not put the server Google key in client code (gotcha 10).
