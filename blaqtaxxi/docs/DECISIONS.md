# DECISIONS.md — questions answered up front

Purpose: Claude should never have to ask a question that is answered here. Every line is tagged:

- **stated** — Steve said it in the discovery notes
- **inferred** — a reasonable reading of the notes or project files (confirm when convenient)
- **assumed** — a default chosen so the build can move; cheap to change; lives in a config file where possible

Confidence: H / M / L. "Where it lives" is the single place to change it.

## A. Product and operating model

| ID | Question | Decision | Tag | Conf | Where it lives |
|---|---|---|---|---|---|
| D-01 | Build number? | **032**. Confirmed by Steve | stated | H | `build.config.ts` |
| D-02 | Repo / domain? | Production domain **`www.blaqtaxxi.com`** (stated). Repo `elwoodberry3/blaqtaxxi` for now; who owns the repo, domain/DNS, Vercel, Neon, Google and Stripe accounts after handoff is **open** (U-C1). Staging URL TBD | stated / open | M | `build.config.ts` |
| D-03 | Category? | Agentic Service Operations (same family as FurnitureHogs, Patient Journey) | assumed | M | `build.config.ts` |
| D-04 | Starting shell? | Clone **`ias-build-template`** (Build 028), then **re-skin to the BLAQ tokens** and remove IAS colors/fonts from the app | inferred | H | Phase 0 |
| D-05 | Fleet size? | **1 driver, N cars** (one car today: a black Nissan Sentra). Only one ride active at a time because there is one driver. Each car has its own price configuration. "Many cars" is read as *one driver, several cars* (U-V1); additional drivers would be a different product (dispatch) and are out of scope | stated (cars) / assumed (reading) | M | `lib/db/schema.ts` |
| D-06 | Service area? | DFW. Out-of-area pickup or drop-off → "Outside service area" message with a request-a-quote TodoChip, no booking | stated / assumed | H | `lib/config/service-area.ts` |
| D-07 | Pricing rule? | **$20 flat** for reference trip ≤30 min. **$25** for >30–45 min. **$30** for >45 min. Price is computed from the **off-peak reference duration** so it stays a true flat fee; rush hour changes *scheduling*, never *price*. Price locked at booking | stated (tiers confirmed for MVP) | H | `lib/pricing/tiers.ts` |
| D-08 | "Louisville, Lil' M … $25"? | **Lewisville, TX → Dallas = $25.** Confirmed by Steve (annotation A1). Pricing test case P1 | stated | H | test fixture |
| D-09 | Availability window and cutoff? | Default day 05:00–24:00 America/Chicago. **Cutoff rule: the driver's end time is the last time a ride can be *booked to start*.** No pickup may be scheduled after it (a pickup exactly at the cutoff is allowed). A ride that starts before the cutoff may finish after it. Same rule for "ride now". Example: a shortened Thursday ending 14:00 → last bookable pickup 14:00. *This is my reading of Steve's statement (pickup start time, not the moment the rider taps Book); confirm* | stated (reading to confirm) | M | `lib/config/availability.ts` |
| D-10 | Lead time? | Scheduled rides: minimum **60 min** ahead. "Ride now" allowed **only** if feasible against current driver position. Max horizon **14 days** | inferred | M | `lib/config/booking.ts` |
| D-11 | Party size? | 1–4 riders, 1 vehicle | assumed | L | `lib/config/booking.ts` |
| D-12 | Driver's first pickup of the day starts from where? | Configurable **home base** (fake address in demo) | assumed | M | `lib/config/driver.ts` |
| D-13 | What does "fill his day" mean in MVP? | Dynamic feasibility slots (the "right around the corner, no need to wait an hour" case) + **"tight fit"** badge on slots that add a ride with minimal dead time. V2: suggest nearby alternate times to riders, waitlist auto-fill on cancellations | assumed | M | `docs/PRD.md` |

## B. Scheduling engine

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-20 | Pickup time semantics? | Rider books an **exact pickup time**. Driver's target is to arrive at or before it. Late-arrival grace shown to rider is 0; driver alerted at risk | assumed | M | `SCHEDULER_SPEC.md` |
| D-21 | Buffer between rides? | **5 min** safety buffer + **3 min** load dwell at pickup + **2 min** unload dwell at drop-off | assumed | M | `lib/config/scheduler.ts` |
| D-22 | Slot granularity? | 5 minutes | assumed | H | `lib/config/scheduler.ts` |
| D-23 | Travel-time source in prod? | **Google Routes API** with traffic-aware routing and a future `departureTime` (predictive traffic). Mapbox is the documented alternative behind the same interface | assumed | M | `lib/routing/` |
| D-24 | Travel time in demo mode? | Deterministic model: haversine × 1.35 road factor ÷ speed profile by hour. Peak (06:30–09:30, 15:30–19:00 local) uses a lower speed. **Labeled illustrative**, calibrated by tests, not a claim about real traffic | assumed | H | `lib/routing/demo.ts` |
| D-25 | Late-risk rule? | Recompute on every driver ping. If projected arrival at next pickup exceeds pickup time → driver alert + async rider notice. If the *following* pickup is also threatened, flag the cascade | assumed | H | `SCHEDULER_SPEC.md` §6 |
| D-26 | Reschedule vs cancel when driver runs late? | MVP: notify + let rider keep or cancel with full refund. No auto-reshuffle | assumed | M | `lib/policy/` |
| D-27 | Concurrency (two riders grab the same slot)? | Insert inside a DB transaction that **re-runs feasibility**; loser gets "slot just taken" with fresh alternatives. Slot holds last **5 min** during checkout | assumed | H | `ARCHITECTURE.md` |

## C. Rider experience

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-30 | Customer accounts? | **No accounts.** The customer books with name, phone, email, pays, and receives a link to `www.blaqtaxxi.com/pickup/…` (before / during / after the trip). See D-85 for the link format | stated | H | `lib/links/` |
| D-31 | What does the rider see over time? | ≥T-30: countdown ("in 32 h 10 m"). T-30 → pickup: live driver location + ETA. In progress: trip status. After: receipt | stated | H | `ARCHITECTURE.md` state machine |
| D-32 | ETA refresh? | Rider page polls every **10 s** inside T-30 window, every 60 s otherwise | assumed | H | `lib/config/tracking.ts` |
| D-33 | Privacy? | A rider never sees another rider's address, name, or time. Location is only exposed from T-30 to ride end, then purged. Pings kept 24 h max | assumed | H | `lib/tracking/` |
| D-34 | Vehicle description? | Shown from the **car record** (photo, color, make, model, plate) that the owner edits in admin. Nothing car-specific committed to the repo. The layout's driver card reads the assigned car | stated | H | `vehicles` table, driver-card |

## D. Driver experience

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-40 | Driver and admin clients? | **Admin backend** at `/admin` (secure login) and **driver console** at `/drive` (mobile-first). Same person today; separate roles in code. Whether `/drive` can be a pure PWA or needs a native shell is **open** and settled by spike SP-1 (U-D1) | stated / open | M | `app/(driver)`, `app/(admin)` |
| D-41 | How does he "drop his location"? | Foreground `watchPosition` (throttled 15–30 s) plus a ping on each status tap, **if a PWA suffices**. *Verified:* Google's Navigation SDK is native-only (no web) and Maps URLs launch navigation with no key. *Unverified:* a backgrounded PWA stops reporting, which would freeze the customer's live map while he uses Google Maps. Spike SP-1 decides; fallback is a native shell for `/drive` only | assumed / open | L | `.claude/skills/realtime-tracking` |
| D-42 | Driver status buttons? | `Heading to pickup` → `Arrived` → `Rider in car` → `Complete` (+ `No-show`) | assumed | H | state machine |
| D-43 | Can driver block time? | Yes: one-off blocks (lunch, car wash). Blocks are just non-ride events in the same feasibility engine | assumed | M | scheduler |
| D-44 | Different hours on a specific day? | **Yes.** The driver can override the start and/or end time, or close the day, for any date. An override beats the default day. Stored as `availability_overrides` (driver_id, local_date, start_min, end_min, closed). Example: a Thursday that ends at 14:00 | stated | H | `lib/db/schema.ts`, `lib/config/availability.ts` |
| D-45 | Driver shortens a day that already has bookings? | Existing confirmed bookings are **honored**; the cutoff only blocks *new* bookings. The console warns and lists bookings past the new cutoff so the driver can decide whether to cancel them (full refund, D-52). Never auto-cancel | assumed | M | `app/api/driver/availability` |

## E. Payments and policy

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-50 | Payment flow? | Book → pay (Stripe PaymentIntent, **test mode**) → confirmed. Flat fee, no tip in MVP | stated | H | `lib/payments/` |
| D-51 | Other payment methods? | Out of scope for MVP. Cash App / Zelle noted as a TodoChip (riders commonly ask) | assumed | M | PRD |
| D-52 | Cancellation policy? | **Modeled on Uber/Lyft, since riders already know them.** Free cancel until **60 min before pickup** (Lyft Scheduled rides charge a fee when cancelled within 1 hour of pickup with a driver matched; verified on Lyft's help page). Inside 60 min, driver not yet en route: flat **$5** fee. After the driver taps "Heading to pickup": flat **$10** fee. Fee never exceeds the fare paid. Driver-initiated cancel: **100%** refund. **Dollar amounts are assumed**: Lyft states fees vary with demand and publishes no fixed amounts, and I could not read Uber's pages (404), so there was nothing to average | 60-min rule verified for Lyft; amounts assumed | M | `lib/policy/policy.config.ts` |
| D-53 | No-show? | Driver taps "Arrived", contacts the rider, waits **5 min**, then may mark no-show. Fee = the **$10** en-route fee (same as Uber/Lyft, where a no-show is a cancellation fee, not a forfeited fare). Lyft's page says a no-show needs "arrived, waited the required time, attempted to contact" but gives no minutes; the **5 min is assumed** | assumed | M | `lib/policy/policy.config.ts` |
| D-54 | Refund mechanics? | Stripe refunds via API; demo mode logs a simulated refund | assumed | H | `lib/payments/` |
| D-55 | Rider cancels because the driver is very late? | If projected lateness is **≥10 min**, the rider may cancel with a **100%** refund. Wait-time fees (Lyft charges per minute after 2 min on standard rides) are **not** in the MVP; TodoChip | assumed | M | `lib/policy/policy.config.ts` |

## F. Stack and integrations

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-60 | Database? | Neon Postgres + Drizzle ORM (already planned across IAS builds) | inferred | H | `lib/db/` |
| D-61 | Live location store? | Upstash Redis (already in stack for rate limiting): last ping per driver with TTL. Postgres holds durable bookings/events only | assumed | H | `lib/tracking/` |
| D-62 | Email? | Resend, per-surface from-address on the verified IAS domain | inferred | H | `lib/notify/` |
| D-63 | SMS? | Twilio behind a seam, **stubbed in demo mode**. Note: US SMS requires A2P 10DLC registration, which has lead time. Email + web-push are the MVP fallbacks | assumed | M | `lib/notify/` |
| D-64 | n8n's job? | Async layer only: booking-confirmed, T-30 reminder, late-risk notice, no-show follow-up, Slack digest, optional HubSpot sync | stated (pattern) | H | `n8n/workflows/` |
| D-65 | Maps and address entry? | **Google Maps** is the map provider (stated). Places Autocomplete restricted to DFW bounds; Routes for travel time; Maps JS for the customer map. Demo mode: fixed fake places and a labeled static map. Browser and server keys are separate (D-102) | stated / assumed | H | `lib/routing/`, `components/MapView` |
| D-66 | Test framework? | Vitest (unit, scheduler) + Playwright (one happy-path e2e) | assumed | H | `package.json` |
| D-67 | Package manager? | npm (lowest friction for Skool students) | assumed | H | repo |

## G. Brand and content

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-70 | Whose brand? | **The client's BLAQ palette** from the supplied layout system (`blaq-*` tokens). It replaces the earlier IAS-token assumption in the app. IAS branding appears only on course material | stated (layout system) | H | `docs/BRAND.md` |
| D-71 | Real data and the course? | **No real customer data in the repo, fixtures, screenshots, or on camera.** The course is recorded on `demo`/`staging` with fake data. Production data lives only in the production database | assumed | H | fixtures, environments |
| D-72 | Episode format? | Build First → Publish Second. 30–50 s clip candidates per phase. **Production wins over teaching** when they conflict. No episode shows client data or is published without client consent (D-101) | stated (pattern) | H | `BUILD_PLAN.md` |
| D-73 | Warning/late color? | The BLAQ palette has red (`blaq-red`) but reserves it for **destructive/negative** actions, so lateness cannot use it. No warning color exists. Until the client approves one: navy text + icon + label ("Running about 5 min late") | open | H | `docs/BRAND.md`, U-B1 |

---

## H. Vehicles and pricing (new)

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-80 | How are cars modeled? | `vehicles` table (label, year/make/model/color, plate, seats, photo, active, `price_config_id`). Admin adds/edits cars | stated (cars, per-car pricing) | H | `lib/db/schema.ts` |
| D-81 | What is a price configuration? | Ordered tiers of `upToMinutes` (a number, or null for no upper bound) and `cents`, evaluated on the **off-peak reference trip time**; first tier with `upToMinutes >= minutes` wins. Defaults $20 / $25 / $30. Editable per car in admin | stated (tiers) / assumed (shape) | M | `lib/pricing/`, `docs/fixtures/vehicles.json` |
| D-82 | Does the customer choose the car? | **No.** The admin assigns the active car; the customer sees the assigned car and one flat price. A customer vehicle choice is a **feature flag, default off** (U-V2), an exception to the layout system's "no ride-tier selector" rule | assumed | M | `lib/config/features.ts` |
| D-83 | What is the scheduling resource? | The **driver**. A car assignment sets price and display, not feasibility | assumed | M | `SCHEDULER_SPEC.md` |
| D-84 | How does a car get assigned? | `vehicle_assignments` windows set in admin (default: the one active car). A booking snapshots the car and price-config version at booking time. If the owner later swaps the car for that date, the price stays locked and the customer is notified | assumed | M | admin A5 |

## I. Customer link and trip page (new)

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-85 | Link format? | Stated: `www.blaqtaxxi.com/pickup/` + customer **last name + last four digits of phone**. **Recommended change (needs approval, U-L1):** keep that as the readable prefix and add a random suffix, e.g. `/pickup/johnson-4821-k9Xp2mQz`, per trip, hashed at rest. **Never authorize on last name + last 4 alone** (about 10,000 possibilities per surname; exposes address, trip, live driver location) | stated + recommended deviation | M | `lib/links/` |
| D-86 | What does the page show and when? | Derived phases: **before** (details, countdown, "check back 30 minutes before"), **approaching** (from T-30: map, driver card, ETA), **in trip** (car on route, estimated arrival), **after** (trip log, completion receipt). Plus cancelled / no-show / expired | stated | H | `SCREEN_MAP.md` C6–C9 |
| D-87 | Lost link? | "Find my ride": last name + last 4 + one-time code by text/email. Rate-limited | assumed | M | `lib/links/` |
| D-88 | Link lifetime? | Valid through **90 days after the trip** (so the receipt and log stay reachable), then contact fields are deleted per the retention default | assumed | M | `lib/links/`, U-C4 |
| D-89 | Normalizing surnames? | Lowercase ASCII slug; strip punctuation (O'Brien → obrien), keep hyphens, transliterate accents, handle multi-word (De La Cruz → delacruz). Store the display name separately | assumed | H | `lib/links/` |

## J. Driver navigation (new)

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-90 | Default navigation? | **Google Maps** via deep link: `https://www.google.com/maps/dir/?api=1&destination=<lat>,<lng>&travelmode=driving&dir_action=navigate` (format and no-key behavior verified against Google's docs). Other apps as a later setting (U-D2) | stated + verified | H | `lib/navigation/` |
| D-91 | After the customer is in the car? | On **Rider in car** the primary action becomes **Navigate to drop-off** (same deep-link builder), with route ETA and **Complete** | stated | H | `SCREEN_MAP.md` D5 |
| D-92 | Confirming the customer at pickup? | Driver asks the customer's last name; a 4-digit code shown by the customer is an optional later feature (U-D3) | assumed | M | D4 |
| D-93 | What does the customer see during the trip? | Google Map with the car on the route, drop-off pin, estimated arrival. Polls every 10 s. Depends on U-D1 | stated | M | C8 |
| D-94 | Message the driver? | Replaced by **tap-to-call/text** using the number set in admin. Number masking is a flagged option (U-R2). No in-app chat | assumed | M | C7 |

## K. Admin backend (new)

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-95 | Access? | `/admin`, Auth.js v5, roles `admin` and `driver` (one person holds both), **2FA or passkey required before production** (U-A1). Every admin route authorizes on the server | stated (secure backend) / assumed (method) | M | `lib/auth/` |
| D-96 | What does admin configure? | Calendar and availability (default + per-date overrides), blocks, cars, per-car price configs, cancellation/no-show policy, service area, lead time/horizon, buffers/dwell, ride-now on/off, message templates, business profile (name, phone, logo, receipt fields), driver display name/photo/contact | stated (calendar, cars, prices, "other relevant things") | H | `SCREEN_MAP.md` §C |
| D-97 | Change control? | Price configs and policies are **versioned** with effective dates; every config write goes to an append-only `config_audit_log`; an existing booking never changes when config changes | assumed | H | `admin-backend` skill |
| D-98 | Two-device edits? | Optimistic concurrency (`version` column); stale edits fail visibly | assumed | H | admin routes |

## L. Production and client (new)

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-99 | Environments? | `demo` (course, fake, zero credentials), `staging` (client review, Stripe test), `production`. Production fails loudly at boot if any required integration is missing; demo fallbacks never run in production | stated (production-grade) / assumed | H | `lib/config/env.ts` |
| D-100 | Go-live gate? | Nothing goes live without written sign-off from Steve **and** the client on `docs/LAUNCH_CHECKLIST.md` (created in Phase 9) and `COMPLIANCE_CHECKLIST.md` | assumed | H | `BUILD_PLAN.md` Phase 9 |
| D-101 | Publishing? | No episode shows client data, and no case study is published, without the client's consent (U-C1) | assumed | H | course process |
| D-102 | Google keys? | Browser key (Maps JS, HTTP-referrer restricted) and server key (Routes, restricted). Server key never reaches the client | assumed | H | `.env.example` |
| D-103 | Stripe ownership? | The client owns the live Stripe account; agency credentials are for test mode only (U-P2) | assumed | H | Phase 9 |
| D-104 | Chips? | `TodoChip`/`StatusChip` render only when `NEXT_PUBLIC_SHOW_TODO_CHIPS=true` (demo/staging). A production build renders none, and CI fails if a chip maps to a launch blocker | assumed | H | `components/` |
| D-105 | Rating? | Keep the layout's 5-star rating as **optional and private to the owner**; never shown publicly (U-R1) | assumed | M | C9 |
| D-106 | Layout deviations? | Recorded in `docs/LAYOUT_AMENDMENTS.md`: no account screens (`profile`, `bottom-nav`, saved places), gray text uses `blaq-gray-text`, one flat-fare line, aria/`dvh` fixes | inferred | H | `LAYOUT_AMENDMENTS.md` |

---

## Open

**See `docs/UNKNOWNS.md`.** It is the single, current list, ordered by what blocks what. Items marked BLOCKING there are never assumed.
