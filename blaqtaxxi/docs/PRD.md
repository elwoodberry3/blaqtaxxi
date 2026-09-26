# PRD — BLAQTAXXI

**Production client build.** The client is the owner-driver of BLAQTAXXI: one driver, one car today, possibly many cars later, each with its own price configuration.

## 1. Problem

A solo driver books rides by DM: "hit me up an hour or two in advance." Each ride is a chain of messages, he tracks where he'll be after each drop-off in his head, and the next customer cannot tell where he is. Pricing is already simple ($20 flat; $25–$30 for long trips). The hard part is **scheduling around one physical driver in real traffic**, and giving both sides a Lyft-like experience without a company behind it.

## 2. Who uses it, and what each side sees

| | **Admin (owner)** | **Driver** | **Customer** |
|---|---|---|---|
| Surface | `/admin`, secure login | `/drive`, mobile | `/book`, then `/pickup/[link]` |
| Account | Yes (2FA/passkey) | Same person | **None** |
| Purpose | Configure what customers see and what the engine allows | Run the day like a Lyft driver | Reserve, pay, follow the trip, keep a receipt |

### Customer journey (link-based)

| Stage | What they see |
|---|---|
| **Book** | Pickup, drop-off, date → dynamic times → one flat fare for the assigned car, cancellation terms → pay |
| **Booked** | Confirmation and the link `www.blaqtaxxi.com/pickup/…`, sent to them and shown on screen |
| **Before the trip** | Trip details, car, price paid, countdown ("in 32 h 10 m"), "check back 30 minutes before your trip to see where your driver is" |
| **Approaching (from T-30)** | Map with the driver, driver card, ETA, tap-to-call/text |
| **During** | Google Map with the car on the route and an estimated arrival |
| **After** | Trip log and completion receipt, optional private rating |

### Driver journey (Lyft-driver view)

Today's schedule → next pickup (time to spare, late-risk) → **Navigate** (Google Maps by default) → **Arrived** → confirm the customer → **Rider in car** → the console now shows **where to go** (Navigate to drop-off) → **Complete** → next pickup.

### Admin controls (drive what customers see)

Calendar and availability (default 05:00–24:00, per-date overrides, closures), time blocks, cars (details, photo, plate, active, when each is assigned), per-car price configuration, cancellation/no-show policy, service area, lead time and horizon, buffers, ride-now on/off, message templates, business profile (name, phone, logo, receipt details), driver display info, bookings with cancel/refund, audit log, account security.

## 3. Scope

### MVP (launch)
1. Customer booking with dynamic slots, per-car flat pricing, Stripe payment, and the customer trip page (before / approaching / during / after).
2. Link-based access with a secure link scheme and recovery.
3. Driver console with schedule, Google Maps navigation handoff, status flow, location drop, late-risk alerts.
4. Admin backend for the controls above, versioned config, audit log, secure sign-in.
5. Scheduling engine (pure module): rush-hour-aware feasibility, dynamic slots, late-risk, cutoff and per-date overrides.
6. Async messaging via n8n: confirmation with link, T-30 reminder, late notice, no-show, receipt.
7. Cancellation/no-show policy modeled on Uber/Lyft, config-driven and versioned.
8. Production readiness: environments, monitoring, backups, launch checklist.

### Later (documented, not built)
- Customer vehicle choice (feature flag, default off).
- Nudges ("book 9:05 instead of 9:00 and it's guaranteed"), waitlist fill on cancellations.
- Tips, promo codes, Cash App/Zelle, recurring rides, round trips.
- 4-digit pickup code, masked driver phone, Waze/Apple Maps setting.

### Out of scope
- More than one driver / dispatch matching, surge pricing, in-app chat, customer accounts, saved places, ratings shown publicly, airport-specific logic (until compliance is verified).

## 4. User stories and acceptance

| # | Story | Acceptance (testable) |
|---|---|---|
| U1 | Customers see only times the driver can make | Slots come from `findSlots()`; none violates the feasibility rule (property test) |
| U2 | After a ride ends near my pickup I can book soon after | Ride ending Downtown, Downtown pickup 15+ min later offered (S3) |
| U3 | A rush-hour 8 am trip blocks the return ride it would strand | S2 |
| U4 | I pay once and the price never changes | Quote locked at checkout; same at any hour (P3); later price edits do not change it (P7) |
| U5 | With a pickup tomorrow I see a countdown, not a map | Trip page shows "in 32 h 10 m" until T-30 |
| U6 | Thirty minutes before pickup I see where he is and how far | Location/ETA only inside T-30; before that `403 not_yet_visible` |
| U7 | During the trip I see the car on the route and my estimated arrival | In-trip payload has car position, route, ETA; stale pings flagged |
| U8 | After the trip I get a log and a receipt | Receipt shows only real data (flat fee, total, business details) |
| U9 | My trip link cannot be guessed | Last name + last 4 alone never authorizes; brute force locks out (link tests) |
| U10 | I can recover a lost link | Recovery requires a one-time code; rate-limited |
| U11 | The driver sees his day in order with gaps and drive times | Timeline from the engine |
| U12 | The driver gets directions to the pickup, then to the drop-off | Deep links built from stored coordinates, tested; primary action changes on **Rider in car** |
| U13 | The driver is warned early when he'll be late | Ping that makes projected arrival exceed pickup raises an alert (L1) |
| U14 | The owner sets hours once and overrides any date | Default 05:00–24:00; overrides beat it; end time limits **starts** (S5, S11a–c) |
| U15 | The owner blocks time | Blocks remove overlapping slots via the same rule (S8) |
| U16 | The owner manages cars and each car's prices | Per-car price config, versioned; assigned car sets the fare and what the customer sees (P6) |
| U17 | Config changes never rewrite existing bookings | Price/policy/copy edits leave existing bookings and sent messages unchanged |
| U18 | Only the owner can change config | Every `/api/admin/*` route returns 403 for non-admins; audit log records each change |
| U19 | Cancel and no-show follow a written policy | C1–C7 |
| U20 | Nobody sees unfinished work in production | Production build renders zero chips; CI check |

## 5. Build-level success measures

Measures of the *build*; no revenue, ride-count, or rating targets are claimed.

- Scheduler, pricing, policy, and link test suites cover every spec case and pass.
- Runs end to end on `demo` with zero credentials; production refuses to boot without real integrations.
- Playwright happy paths for customer and admin.
- `/brand-audit` clean; measured contrast passes AA.
- Launch checklist complete with evidence and written sign-off.

## 6. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| **Live location while the driver uses Google Maps** (Navigation SDK is native-only; a backgrounded PWA likely stops reporting) | Customer's live map freezes mid-drive | Spike SP-1 on the client's phone before building tracking; native shell for `/drive` if needed (U-D1) |
| **Guessable customer link** (last name + last 4) | Privacy breach: pickup address, trip, live location | Random suffix, hashing, rate limits, expiry (U-L1) |
| Routing cost and accuracy | Bills; wrong ETAs | Cache by cell + hour, budget alerts, buffer/peak factors tuned from real trips (SP-3) |
| Rush-hour under-estimation | Late pickups | Tunable buffer and factors; late-risk alerts; calibration |
| **Regulatory, insurance, permits, privacy** | Cannot legally launch | **Unverified.** Launch gate in `COMPLIANCE_CHECKLIST.md` |
| SMS registration lead time | No texts at launch | Email + on-screen link at launch; SMS behind a seam (U-N1) |
| Payment fees on a $20 fare | Thin margin | Decide who absorbs fees (U-P1); verify current Stripe pricing |
| Ownership and consent unclear | Cannot deploy or publish | U-C1 answered before publishing or Phase 9 |
| Contrast failures in the supplied layouts | Accessibility failures | Fixes in `BRAND.md`; approval of `blaq-gray-text` (U-B1) |
| Run costs and support after handoff | Client outage or surprise bills | Budget alerts, runbook, agreed support (U-C2, U-C3) |

## 7. One Build, Three Doors

- **Employment:** a real-time scheduling engine, versioned admin config, secure link design, and an async layer; a production-grade case study (with client consent).
- **Consulting:** `lib/scheduler`, `lib/routing`, and the admin-config pattern generalize to any solo operator who travels between appointments.
- **Brand:** a live-build series with honest spikes and audits.
