# PRD — BLAQTAXXI

**Production client build.** The client is the owner-driver of BLAQTAXXI: one driver, one car today, possibly many cars later, each with its own price configuration.

## 1. Problem

A solo driver books rides by DM: "hit me up an hour or two in advance." Each ride is a chain of messages, he tracks where he'll be after each drop-off in his head, and the next customer cannot tell where he is. Pricing is already simple ($20 flat; $25–$30 for long trips). The hard part is **scheduling around one physical driver in real traffic**, and giving both sides a Lyft-like experience without a company behind it.

## 1b. Delivery model: pilot, then handoff

1. **Pilot** on IAS's infrastructure (`blaqtaxxi.iasbootcamp.com`). The client uses it with written kick-the-tires instructions for **1–2 feedback rounds** to tune it to his exact needs. The public may use it too, **up to the moment money would be paid**; no payment is taken.
2. **Approval.** The client approves the final in writing.
3. **Handoff.** IAS gives him instructions to deploy the same app on his own domain, Vercel, Google Cloud, Stripe, and other accounts. Live payments exist only there.

See `docs/PILOT_PLAYBOOK.md` and `docs/CLIENT_DEPLOY_GUIDE.md`.

## 2. Who uses it, and what each side sees

| | **Admin (owner)** | **Driver** | **Customer** |
|---|---|---|---|
| Surface | `/admin`, secure login | `/drive` in a thin native shell (iPhone default, Android supported) | `/book`, then `/pickup/[link]` |
| Account | Yes (2FA/passkey) | Same person | **None** |
| Purpose | Configure what customers see and what the engine allows | Run the day like a Lyft driver | Reserve, pay, follow the trip, keep a receipt |

### Customer journey (link-based)

| Stage | What they see |
|---|---|
| **Book** | Pickup, drop-off, date → **choose a car** (photo, seats, that car's price) → times for that car → one flat fare, cancellation terms → pay |
| **Booked** | Confirmation and the link `www.blaqtaxxi.com/pickup/…`, sent to them and shown on screen |
| **Before the trip** | Trip details, car, price paid, countdown ("in 32 h 10 m"), "check back 30 minutes before your trip to see where your driver is" |
| **Approaching (from T-30)** | Map with the driver, driver card, ETA, tap-to-call/text |
| **During** | Google Map with the car on the route and an estimated arrival |
| **After** | Trip log and completion receipt, optional private rating |

### Driver journey (Lyft-driver view)

Today's schedule → next pickup (time to spare, late-risk) → **Navigate** (Google Maps by default) → **Arrived** → confirm the customer → **Rider in car** → the console now shows **where to go** (Navigate to drop-off) → **Complete** → next pickup.

### Admin controls (drive what customers see)

Calendar and availability (default 05:00–24:00, per-date overrides, closures), time blocks, cars (details, uploaded photos, plate, active, **when each is assigned**, switch-car swap blocks), per-car price configuration, cancellation/no-show policy, service area, lead time and horizon, buffers, ride-now on/off, message templates, business profile (name, phone, logo, receipt details), driver display info, bookings with cancel/refund, audit log, account security, **config export/import**, pilot feedback inbox.

## 3. Scope

### MVP (launch)
1. Customer booking with **car choice**, dynamic slots per car, per-car flat pricing, Stripe payment, and the customer trip page (before / approaching / during / after).
2. Link-based access with a secure link scheme and recovery.
3. Driver console with schedule, Google Maps navigation handoff, status flow, location drop, late-risk alerts.
4. Admin backend for the controls above, versioned config, audit log, secure sign-in.
5. Scheduling engine (pure module): rush-hour-aware feasibility, dynamic slots, late-risk, cutoff and per-date overrides.
6. Async messaging via n8n: confirmation with link, T-30 reminder, late notice, no-show, receipt.
7. Cancellation/no-show policy modeled on Uber/Lyft, config-driven and versioned.
8. Production readiness: environments, monitoring, backups, launch checklist.

### Later (documented, not built)
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
| U16 | The owner manages cars, when each is assigned, and each car's prices | Per-car price config, versioned; assignment windows without overlap; a car switch creates a swap block (S12) |
| U21 | The customer chooses the car | Car list shows only cars with a slot and enough seats, each with its own price for the trip (P6, S12d); a car not assigned at that time is never offered (S12a) |
| U22 | The client can try everything in the pilot, and the public can go up to payment | Pilot banner everywhere; only allow-listed testers can complete a test payment; unpaid-hold data is purged within 24 h (D-107, D-108) |
| U23 | The driver can change phones | New-device sign-in re-authenticates and revokes the old device; pings from a revoked device are rejected (D-110) |
| U24 | The client can move his configuration to his own deployment | Config export/import round-trips hours, cars, prices, policy, templates, profile; never bookings or customer data (D-116) |
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
| **Live location while the driver uses Google Maps** (Navigation SDK is native-only, so the driver app opens Google Maps and is backgrounded) | Customer's live map could freeze mid-drive | The approved native shell keeps sending location; spike SP-1 on a real iPhone and a real Android phone before Phase 5 (U-D7); plugin behavior and testing-track rules are unverified (U-D6) |
| Public data in the pilot | Real names/phones/addresses before payment | Purge unpaid holds within 24 h; pilot notice and terms (D-107, U-G1) |
| IAS handling the client's real fares | Money-handling and legal exposure | Not done: pilot is test mode; the client uses his own live Stripe account (D-119, U-P2) |
| **Guessable customer link** (last name + last 4) | Privacy breach: pickup address, trip, live location | Random suffix, hashing, rate limits, expiry (U-L1) |
| Routing cost and accuracy | Bills; wrong ETAs | Cache by cell + hour, budget alerts, buffer/peak factors tuned from real trips (SP-3) |
| Rush-hour under-estimation | Late pickups | Tunable buffer and factors; late-risk alerts; calibration |
| **Regulatory, insurance, permits, privacy** | Cannot legally launch | **Unverified.** Launch gate in `COMPLIANCE_CHECKLIST.md` |
| SMS registration lead time | No texts at launch | Email + on-screen link at launch; SMS behind a seam (U-N1) |
| Payment fees on a $20 fare | Thin margin | Decide who absorbs fees (U-P1); verify current Stripe pricing |
| Ownership and consent unclear | Cannot deploy or publish | U-C1 answered before publishing; handoff terms (U-H1) before Phase 10 |
| Contrast failures in the supplied layouts | Accessibility failures | Fixes in `BRAND.md`; approval of `blaq-gray-text` (U-B1) |
| Run costs and support after handoff | Client outage or surprise bills | Budget alerts, runbook, agreed support (U-C2, U-C3) |

## 7. One Build, Three Doors

- **Employment:** a real-time scheduling engine, versioned admin config, secure link design, and an async layer; a production-grade case study (with client consent).
- **Consulting:** `lib/scheduler`, `lib/routing`, and the admin-config pattern generalize to any solo operator who travels between appointments.
- **Brand:** a live-build series with honest spikes and audits.
