# DECISIONS.md — questions answered up front

Purpose: Claude should never have to ask a question that is answered here. Every line is tagged:

- **stated** — Steve said it in the discovery notes
- **inferred** — a reasonable reading of the notes or project files (confirm when convenient)
- **assumed** — a default chosen so the build can move; cheap to change; lives in a config file where possible

Confidence: H / M / L. "Where it lives" is the single place to change it.

## A. Product and operating model

| ID | Question | Decision | Tag | Conf | Where it lives |
|---|---|---|---|---|---|
| D-01 | Build number? | **032** (`projects.csv` max id is 031) | assumed | M | `build.config.ts` |
| D-02 | Repo / subdomain? | `elwoodberry3/blaqtaxxi` / `blaqtaxxi.elwoodberry.com` (follows existing pattern) | assumed | H | `build.config.ts` |
| D-03 | Category? | Agentic Service Operations (same family as FurnitureHogs, Patient Journey) | assumed | M | `build.config.ts` |
| D-04 | Starting shell? | Clone **`ias-build-template`** (Build 028): one config file drives content, status, repo links, CTAs | inferred | H | Phase 0 |
| D-05 | Fleet size? | Exactly 1 driver, 1 car, 1 active ride. Schema still carries `driver_id` so the engine can be reused for other solo operators | stated / assumed | H | `lib/db/schema.ts` |
| D-06 | Service area? | DFW. Out-of-area pickup or drop-off → "Outside service area" message with a request-a-quote TodoChip, no booking | stated / assumed | H | `lib/config/service-area.ts` |
| D-07 | Pricing rule? | **$20 flat** for reference trip ≤30 min. **$25** for >30–45 min. **$30** for >45 min. Price is computed from the **off-peak reference duration** so it stays a true flat fee; rush hour changes *scheduling*, never *price*. Price locked at booking | stated + tier split assumed | M | `lib/pricing/tiers.ts` |
| D-08 | "Louisville, Lil' M … $25"? | Read as **Lewisville → Dallas = $25** (see annotation A1). Add as a pricing test case | inferred | H | test fixture |
| D-09 | Availability window? | Daily 05:00–24:00 America/Chicago. **A ride must be able to *finish* by 24:00** (safer default). Configurable to "pickup must start by 24:00" | stated + boundary assumed | M | `lib/config/availability.ts` |
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
| D-30 | Rider accounts? | **No accounts.** Book with name + phone + email. Manage via a signed magic link (`/ride/[token]`) | assumed | M | `lib/tokens.ts` |
| D-31 | What does the rider see over time? | ≥T-30: countdown ("in 32 h 10 m"). T-30 → pickup: live driver location + ETA. In progress: trip status. After: receipt | stated | H | `ARCHITECTURE.md` state machine |
| D-32 | ETA refresh? | Rider page polls every **10 s** inside T-30 window, every 60 s otherwise | assumed | H | `lib/config/tracking.ts` |
| D-33 | Privacy? | A rider never sees another rider's address, name, or time. Location is only exposed from T-30 to ride end, then purged. Pings kept 24 h max | assumed | H | `lib/tracking/` |
| D-34 | Vehicle description? | "Black Nissan" only. No plate or model printed in repo/demo | inferred | M | copy deck |

## D. Driver experience

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-40 | Driver client? | Mobile-first **PWA** at `/drive`. Auth.js v5 with a single-email allowlist | assumed | H | `app/(driver)/` |
| D-41 | How does he "drop his location"? | Foreground `watchPosition` while console is open (throttled 15–30 s) **plus** a ping on every status tap. Not background tracking (iOS limit) | assumed | H | `.claude/skills/realtime-tracking` |
| D-42 | Driver status buttons? | `Heading to pickup` → `Arrived` → `Rider in car` → `Complete` (+ `No-show`) | assumed | H | state machine |
| D-43 | Can driver block time? | Yes: one-off blocks (lunch, car wash). Blocks are just non-ride events in the same feasibility engine | assumed | M | scheduler |

## E. Payments and policy

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-50 | Payment flow? | Book → pay (Stripe PaymentIntent, **test mode**) → confirmed. Flat fee, no tip in MVP | stated | H | `lib/payments/` |
| D-51 | Other payment methods? | Out of scope for MVP. Cash App / Zelle noted as a TodoChip (riders commonly ask) | assumed | M | PRD |
| D-52 | Cancellation policy? | Free until **T-60 min**. Inside 60 min: **50%** retained. Driver-initiated cancel: **100%** refund | assumed | **L** | `lib/policy/policy.config.ts` |
| D-53 | No-show? | Driver waits **5 min** after "Arrived", then may mark no-show. **100%** fare retained | assumed | **L** | `lib/policy/policy.config.ts` |
| D-54 | Refund mechanics? | Stripe refunds via API; demo mode logs a simulated refund | assumed | H | `lib/payments/` |

## F. Stack and integrations

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-60 | Database? | Neon Postgres + Drizzle ORM (already planned across IAS builds) | inferred | H | `lib/db/` |
| D-61 | Live location store? | Upstash Redis (already in stack for rate limiting): last ping per driver with TTL. Postgres holds durable bookings/events only | assumed | H | `lib/tracking/` |
| D-62 | Email? | Resend, per-surface from-address on the verified IAS domain | inferred | H | `lib/notify/` |
| D-63 | SMS? | Twilio behind a seam, **stubbed in demo mode**. Note: US SMS requires A2P 10DLC registration, which has lead time. Email + web-push are the MVP fallbacks | assumed | M | `lib/notify/` |
| D-64 | n8n's job? | Async layer only: booking-confirmed, T-30 reminder, late-risk notice, no-show follow-up, Slack digest, optional HubSpot sync | stated (pattern) | H | `n8n/workflows/` |
| D-65 | Address entry? | Google Places Autocomplete restricted to DFW bounds; demo mode uses a fixed list of fake DFW landmarks | assumed | M | `lib/routing/places.ts` |
| D-66 | Test framework? | Vitest (unit, scheduler) + Playwright (one happy-path e2e) | assumed | H | `package.json` |
| D-67 | Package manager? | npm (lowest friction for Skool students) | assumed | H | repo |

## G. Brand and content

| ID | Question | Decision | Tag | Conf | Where |
|---|---|---|---|---|---|
| D-70 | Whose brand? | Built as an **IAS demo** using IAS tokens. A `brand.config.ts` seam lets BLAQTAXXI's own palette/logo drop in later without touching components | assumed | M | `docs/BRAND.md` |
| D-71 | Real data in the course? | **No.** Fake riders, fake addresses, fake phone numbers only | assumed | H | fixtures |
| D-72 | Episode format? | Build First → Publish Second. 30–50 s standalone clip candidates per phase; one "Policy to Code" episode on ride-service rules → config | stated (pattern) | H | `BUILD_PLAN.md` |
| D-73 | Warning/error color? | The IAS palette has **none**. Until Steve approves a token pair, late/error states use Ink text + icon + text label on Primary-800. No invented red | open | H | `docs/BRAND.md` |

---

## Open — needs Steve (everything else above is safe to proceed on)

Only items that are **hard to reverse** or **change what gets built** should stop work. In priority order:

1. **Is BLAQTAXXI a real launch or a course demo?** (Drives D-70/D-71 and whether `COMPLIANCE_CHECKLIST.md` is a gate or a lesson.) *Default: course demo.*
2. **Confirm cancellation and no-show policy** (D-52, D-53). *Default above, config-driven.*
3. **Confirm midnight rule** (D-09): must the ride finish by 24:00, or only start by 24:00? *Default: finish.*
4. **Confirm price tiers** (D-07): is >45 min = $30 right, and does pricing follow trip time or zone? *Default above.*
5. **Confirm "Lewisville"** (D-08) and the build number (D-01).
6. **SMS provider and timing** (D-63).

Nothing above blocks Phases 0–2. Phases 3+ can proceed on defaults; Phase 6 (payments) stays in test mode regardless.
