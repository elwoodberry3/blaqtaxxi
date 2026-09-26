---
name: realtime-tracking
description: Use when building the driver app's navigation flow and location drop, the customer trip page at /pickup/[link] (before, approaching, during, after), ETA, the T-30 visibility gate, staleness, device handling, the Google Maps deep-link builder, the customer link scheme, or the driver simulator.
---

# realtime-tracking

Read `docs/ARCHITECTURE.md` §2, §5, §6, §8, `docs/SCREEN_MAP.md` §A–B, `docs/SCHEDULER_SPEC.md` §6, and `docs/UNKNOWNS.md` (SP-1, SP-2, U-D6) first. For the native side, load `driver-shell`.

## Non-negotiables

1. **n8n is never in this path.** Ping → Redis → ETA → customer page, all inside Next.js route handlers.
2. **The T-30 gate is server-enforced.** The API refuses location outside the window (`403 not_yet_visible`). Hiding it in the UI is not a gate. Use the pure `visibilityWindow(pickupAt, now, status)` from `lib/scheduler`.
3. **Honest about freshness.** Show "last seen N s ago", flag pings older than 90 s, never render a stale marker as live.
4. **Privacy by construction.** A customer sees only their own booking and, inside their window, the driver's position. No other customer's data ever crosses the API. Pings are transient (Redis TTL 120 s), never in Postgres.
5. **The driver app does not own the map.** It opens Google Maps. Because Google Maps is then in front, the **native shell** keeps sending location (approved, D-40/D-41). Phase 5 does not start until the SP-1/SP-2 results are recorded.

## Driver app (`/drive` inside the native shell)

States and their one primary action (44 px buttons):

| State | Shows | Primary action |
|---|---|---|
| Idle / between rides | Next pickup, time to spare, late-risk | **Open in Google Maps** to the pickup |
| Heading to pickup | ETA | **Arrived** |
| At pickup | Wait timer (5 min), call/text | **Rider in car** (and **No-show** once the wait elapses) |
| In trip | Route ETA | **Open in Google Maps** to the drop-off, then **Complete** |

- Every button tap sends a location ping and a status transition (`POST /api/driver/status`).
- **`lib/navigation`** builds the deep link; unit-test it (encoding, precision, missing coordinates):
  `https://www.google.com/maps/dir/?api=1&destination=<lat>,<lng>&travelmode=driving&dir_action=navigate`
  (format and no-key behavior verified against Google's Maps URLs docs). Use stored coordinates, not free-text addresses. Google Maps is the default; other apps are a later setting (U-D2). Spike SP-4 confirms it opens and starts navigation on both phones.
- **Devices.** iPhone is the default target and Android is supported (his phone today). Every ping carries `deviceId`. Only the device signed in for the active shift is accepted; signing in on a new device requires re-auth and revokes the old one (D-110). Reject pings from revoked devices.
- Ask the customer's last name at pickup (D-92). Never show the driver a customer's full link secret.
- Permission and battery states (SCREEN_MAP D8) are visible screens, not silent failures.

## Server

- `POST /api/driver/ping`: Zod-validate `{lat,lng,accuracy,at,deviceId}`, reject revoked devices, clamp absurd values, write `driver:{id}:loc` (TTL 120 s), run `projectDay` (cached per ping timestamp), emit a late-risk event **only on change**.
- `GET /api/pickup/[link]`: verify the link (§ below) → compute `visibilityWindow` → return the phase payload: `before` (details, chosen car, countdown), `approaching` (driver position with `seenSecondsAgo`/`stale`, ETA to pickup), `in_trip` (car position, route, ETA to drop-off), `after` (log, receipt). No cancel in trip.
- ETA via `provider.minutes(...)` through the routing cache; memoize per `(bookingId, pingAt)`. Fetch the route polyline once at trip start and store it on the booking.

## Customer trip page (`/pickup/[link]`)

- Load `layout-system` and start from `active-trip` / `fare-summary` (SCREEN_MAP C6–C9). No bottom nav, no account UI. Show the **assigned car** (photo, model, plate) in the driver card.
- Map via a `MapView` component (Google Maps JS, browser key); on `demo` a labeled static map. Always put a **text ETA** next to the map.
- Poll every 60 s before T-30, every 10 s inside the window; stop when done; back off on errors.
- Announce meaningful changes with `aria-live="polite"`, not every tick. Times and prices use tabular figures.
- Copy: "Check back 30 minutes before your trip to see where your driver is." / "Driver is 12 min away."
- Late-risk uses `blaq-amber` with an icon and a label, **never red** (D-73).

## Customer link (`lib/links`) — a bearer secret (approved format)

- `/pickup/<lastname>-<last4>-<random>`, per trip. **Never authorize on last name + last 4 alone.**
- Random part: at least 64 bits from a CSPRNG; store only a hash; constant-time compare; valid until 90 days after the trip; per-IP and per-prefix rate limits with lockout; strict limits on recovery.
- Normalize surnames (O'Brien → obrien, De La Cruz → delacruz, accents, hyphens); store the display name separately.
- Scrub `/pickup/*` from logs and analytics; the page sends `Referrer-Policy: no-referrer` and `noindex`.
- **Tests first:** entropy, collisions (two customers, same surname and last 4), normalization, expiry, brute-force lockout, recovery flow, cross-customer isolation.

## Dev simulator

`/dev/sim` (`demo` only) replays a route from fixtures at 10× speed by POSTing pings so the customer and driver screens are demoable at a desk.

## Tests to write first

V1 (visibility), L1/L2 (late-risk + cascade), API test that no location leaks at T-31 min and is returned at T-29, no cross-customer leakage, stale-ping flag, revoked-device rejection, deep-link builder, link security suite, rate limiting.

## Pitfalls

- `NEXT_PUBLIC_*` read through a module-level const → `undefined` in production. Read inside the function.
- Local times: convert with `America/Chicago` once, in `lib/time.ts`.
- Clock skew: trust server time for windows; use the device timestamp only for "seen N s ago", clamped to ≥ 0.
- No WebSocket/SSE for the MVP (gotcha 7).
- Do not put the server Google key in client code (gotcha 10).
