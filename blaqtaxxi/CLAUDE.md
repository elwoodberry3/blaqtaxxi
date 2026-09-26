@CLAUDE.layout.md

# CLAUDE.md — BLAQTAXXI ("One-to-One Uber")

> Governance file for Claude Code. Read this first, every session. If this file and a chat message disagree, follow the chat message, then tell me so this file can be updated.
> The line above, `@CLAUDE.layout.md`, imports the BLAQ layout/token rules. If your Claude Code version does not expand `@` imports, paste that file's contents under its own heading here.

**This is a production build for a paying client (the owner-driver of BLAQTAXXI).** It is also taught episodically in an advanced Skool course. When those two goals conflict, **production wins**: no real customer data on camera, no unfinished work visible to customers, no shortcuts that would not survive launch.

**Build:** IAS Build 032 (confirmed) · **Production domain:** `www.blaqtaxxi.com` (stated) · **Repo:** `elwoodberry3/blaqtaxxi` (ownership after handoff is open, see `docs/UNKNOWNS.md`)
**Governing principle:** One Build. Three Doors. Every phase yields (1) a shippable piece of the client's product, (2) a reusable consulting module, (3) a recordable episode.

---

## 1. What this product is

One driver. One active fare at a time. Today one car (a black Nissan Sentra); later possibly many cars, each with its own price configuration. Customers **reserve a time**, pay a **flat fee** up front, and follow the driver by link. The driver fills his day with as many rides as physically possible. The core problem is **not dispatch, it is travel-time-aware scheduling**: a booking is only offered if the driver can reach it on time from wherever his *previous ride ends*, in the traffic at that hour, and still make his *next* pickup.

### Three surfaces (each sees something different)

| Surface | Route | Who / auth | What they do |
|---|---|---|---|
| **Admin backend** | `/admin` | The owner-driver. Secure login (method open, U-A1) | Configure calendar and availability, cars, price configs, cancellation policy, service area, notification copy, and everything else that drives what customers see. Review bookings |
| **Driver console** | `/drive` | Same person, mobile-first | Lyft-driver view: today's schedule, **directions to the pickup** (Google Maps by default), confirm the customer is in the car, then **directions to the drop-off**. Status buttons, location drop |
| **Customer** | `/book` → `/pickup/[link]` | No account. Pays, then receives a link | **Before:** trip details, countdown, "check back 30 minutes before". **During:** the trip on a Google Map with live ETA. **After:** trip log and completion receipt |

The customer link is `www.blaqtaxxi.com/pickup/…` built from the customer's **last name and the last four digits of their phone**. **Never authorize access with those two facts alone** (see §7, gotcha 11). The exact format is an open decision.

## 2. Read order (before writing any code)

1. `docs/UNKNOWNS.md` — what is still open, what blocks what. **Do not build on an item marked BLOCKING without asking.**
2. `docs/DECISIONS.md` — answered questions and labeled assumptions. **Do not re-ask anything answered here.**
3. `docs/PRD.md` — scope, what each side sees, user stories.
4. `docs/SCREEN_MAP.md` — every screen, by audience, and whether its layout exists, needs adapting, or is new.
5. `docs/SCHEDULER_SPEC.md` — the heart of the build.
6. `docs/ARCHITECTURE.md` — real-time vs async split, data model, states, API, environments.
7. `docs/LAYOUT_AMENDMENTS.md` — where and why we deviate from the supplied layout system.
8. `docs/BUILD_PLAN.md` — phases, prompts, gates.
9. `docs/BRAND.md` — BLAQ tokens and the contrast findings.
10. `SKILLS.md` — index of `.claude/skills/*`. Load the relevant skill before working in its area. **Before building any screen, load `layout-system` and read its `manifest.json` first.**

## 3. Non-negotiables (hard constraints)

1. **Baseline Stack + DXP.** Next.js 14 (App Router) · TypeScript (strict) · Tailwind · Vercel · cloud-hosted n8n (`https://iautomateshit.app.n8n.cloud`). Composable, decoupled frontend/backend, modular payment/auth/integration subsystems, shared design tokens.
2. **Real-time vs async split.** The live path (pings → ETA → customer screen, booking, slot search) **never touches n8n**. n8n owns only the async layer: confirmations, T-30 reminders, late notices, digests.
3. **The scheduler is a pure module.** `lib/scheduler/**` has **zero** imports from Next, the DB, Stripe, or any network client. Travel time is injected via `TravelTimeProvider`; prices via a `PriceConfig` argument.
4. **Environments.** `demo` (course, fake data, zero credentials), `staging` (client review, Stripe test), `production` (real). Every integration has a demo fallback for `demo` only. **Production fails loudly at boot if any required integration is missing.** The course is recorded on `demo`/`staging`, never on production.
5. **Visible gaps are for demo/staging only.** `TodoChip`/`StatusChip` render only when `NEXT_PUBLIC_SHOW_TODO_CHIPS=true`. A production build must render **zero** chips, and CI must fail if any chip is tied to a launch-blocking item (`docs/UNKNOWNS.md`). Customers never see unfinished work.
6. **No fabricated numbers.** Demo travel times are an illustrative model, labeled as such. No invented revenue, ride counts, ratings, or testimonials anywhere.
7. **No real personal data in the repo, in fixtures, in screenshots, or on camera.** Fixtures use fake customers/addresses. Production customer data lives only in the production database.
8. **Stripe stays in test mode** until the go-live gate in `docs/BUILD_PLAN.md` Phase 9 is signed off in writing. The client, not the agency, must own the live Stripe account (U-P2).
9. **Ask before irreversible moves:** live-mode payments, messaging real people, deleting data, force-pushing, paid-API spend beyond the dev tier, changing production config.
10. **Layout system is the visual baseline.** Use its layouts and `blaq-*` tokens; no raw hex, no arbitrary color values. Deviations only as recorded in `docs/LAYOUT_AMENDMENTS.md`, and **where `CLAUDE.layout.md` conflicts with the amendments (for example gray as text), the amendments win.**
11. **Config drives the customer.** Anything a customer sees that the owner might change (hours, prices, car, policy amounts, copy) comes from admin-managed config, versioned, never hardcoded. A price or policy change never rewrites an existing booking.

## 4. Domain rules you must not get wrong

| Rule | Value | Source |
|---|---|---|
| Drivers / cars | **1 driver**, **N cars** (1 today). Only one ride active at a time | stated |
| Price | **Per car**: each car has its own price configuration. Default tiers: **$20** ≤30 min, **$25** >30–45, **$30** >45 (off-peak reference trip time) | stated (tiers confirmed) |
| Service area | Dallas–Fort Worth | stated |
| Availability | default **05:00–24:00** America/Chicago; override any date (shorten, move, close) | stated |
| Cutoff | the driver's end time is the last time a ride can be **booked to start**; a ride may finish after it | stated (reading to confirm, D-09) |
| Cancellation | free until T-60 min; $5 inside 60 min; $10 once the driver is en route or on a no-show. Config per policy version | Lyft 60-min rule verified; amounts assumed (D-52) |
| Customer visibility | countdown always; **live location + ETA only from T-30 min** | stated |
| Payment | book → pay → flat fee (price locked at booking) | stated |
| Driver navigation | Google Maps by default via deep link; on "Rider in car" the console shows where to go next | stated |
| Pricing vs traffic | rush hour affects **scheduling**, never **price** | assumed (D-07) |

**The one sentence that defines correctness:** *A booking is valid only if `prevRide.end + travel(prev.dropoff → new.pickup, at that time) + buffer ≤ new.pickup` AND `new.end + travel(new.dropoff → next.pickup, at that time) + buffer ≤ next.pickup`, and the pickup is no later than that date's cutoff.* Rush hour makes `travel(...)` bigger; a drop-off "around the corner" makes it tiny, so the next pickup can be minutes later, not an hour later.

## 5. Repo layout (target)

```
app/
  (customer)/book/           slot picker → quote → checkout
  (customer)/pickup/[link]/  before (details, countdown) · during (live map, ETA) · after (log, receipt)
  (driver)/drive/            schedule, navigate-to-pickup, confirm rider, navigate-to-dropoff
  (admin)/admin/             calendar, cars, prices, policy, service area, copy, bookings
  api/                       route handlers (ARCHITECTURE.md)
lib/
  scheduler/                 PURE. feasibility, slot search, late-risk. No I/O
  routing/                   TravelTimeProvider: google.ts | demo.ts
  pricing/                   PriceConfig → quote (per car)
  policy/                    cancellation/no-show rules (versioned)
  tracking/                  ping ingest, T-30 gate
  navigation/                Google Maps deep-link builders
  links/                     customer link generation + verification
  payments/  notify/  db/  auth/  config/
components/                  ui/ (layout-system components), TodoChip, StatusChip
n8n/workflows/  tests/  docs/  .claude/
```

## 6. Coding conventions

- TypeScript `strict`. No `any` in `lib/scheduler`. Discriminated unions for states.
- All times stored **UTC**, displayed **America/Chicago**, via one `lib/time.ts`. DST-day tests.
- Money is **integer cents**.
- **Styling: `blaq-*` Tailwind tokens only** (`blaq-navy`, `blaq-royal`, `blaq-green`, `blaq-red`, `blaq-gray`, `blaq-black`, `blaq-canvas`, `blaq-white`). No hex, no `bg-[#…]`. Red is for destructive/negative only. Contrast findings and the proposed text-gray token are in `docs/BRAND.md`; use them.
- Fonts: the layout system's system font stack unless the client specifies a brand font (U-B3). If a Google font is chosen, load it with a runtime `<link>`, not `next/font/google` (gotcha 3).
- Accessibility: WCAG AA (measured, see BRAND.md), real `<button>`s, focus rings, `aria-live` for ETA, 44 px targets on `/drive` and `/admin`.
- Every route handler validates input with Zod and returns typed errors. Admin routes check role server-side.
- Small commits, one concern each, conventional messages.

## 7. Known gotchas

1. **`output: 'export'` silently kills API routes.** Never set it.
2. **`NEXT_PUBLIC_*` vars are inlined at build time.** Read them inside the function/component body, never through a module-level `const` chain.
3. **`next/font/google` fails at build** when `fonts.googleapis.com` is blocked. Use a runtime `<link>`.
4. **n8n auth:** header key exactly `Authorization`, value includes `Bearer `. Credential *ownership* vs project *sharing* causes silent 401s.
5. **Version drift:** never cherry-pick files from different patch drops. Apply full consolidated patches.
6. **HubSpot Free ceiling:** 10 custom contact properties.
7. **Vercel functions are not long-lived sockets.** Poll the customer page (10 s in window), short POSTs for pings. No WebSocket/SSE for the MVP.
8. **Background location is the biggest technical risk (verified pieces below).**
   - *Verified:* Google's **Navigation SDK exists for Android and iOS (plus Flutter/React Native) only; there is no web version.** A PWA cannot embed Google turn-by-turn. Google Maps **URLs** work with no API key, and `dir_action=navigate` launches turn-by-turn on mobile.
   - *My understanding, NOT yet verified on the client's phone:* when the driver taps through to the Google Maps app, our PWA goes to the background and browsers stop delivering location, so the customer's live map **freezes during the drive**. **Do not build the customer's "during" screen on the assumption that this works.** Run the Phase 0.5 spike first. The decision (PWA vs native shell) is BLOCKING, U-D1.
9. **Routing APIs cost money per element.** Cache by (origin cell, destination cell, hour bucket). Cap candidates per search.
10. **Google Maps keys:** one browser key (Maps JS, HTTP-referrer restricted) and one server key (Routes, restricted). Never ship the server key to the client. Check current Google Maps Platform pricing before launch; not verified here.
11. **The customer link is a bearer secret.** Last name + last four digits of a phone number has roughly 10,000 possibilities per surname, so it is guessable and it exposes a person's pickup address, trip, and the driver's live location. **Default:** human-readable prefix plus an unguessable random suffix (`/pickup/johnson-4821-k9Xp2mQz`), rate-limited, expiring after the trip window. Store only a hash. Normalize surnames (O'Brien, De La Cruz, hyphens). Confirm with the client (U-L1).
12. **Never leak across customers.** One customer's link must never reveal another customer's name, address, time, or the driver's location outside that customer's window.
13. **Price and policy changes are versioned.** A booking snapshots its price-config version, amount, policy version. Editing config creates a new version.

## 8. Working agreement

1. **Repo-first.** Read the actual current files before changing anything.
2. **Structured decisions.** Anything architectural or irreversible that is not in `DECISIONS.md` → stop and ask with `AskUserQuestion` (2–4 options, recommended first). Cheap-to-reverse → decide, log as `assumed`, continue. **Items marked BLOCKING in `UNKNOWNS.md` are never assumed.**
3. **Phase gates.** Follow `docs/BUILD_PLAN.md`; do not start Phase N+1 until N's acceptance checks pass and I say go.
4. **Tests first** for `lib/scheduler`, `lib/pricing`, `lib/policy`, `lib/links`.
5. **Show, don't claim.** Run it and show output. "Should work" is not done.
6. **Screens:** load `layout-system`, read `manifest.json`, match the nearest layout, then build. If no layout fits (many admin/driver screens), say so and build from `components/` primitives and the tokens, then add it to `SCREEN_MAP.md`.
7. **Narrate for the camera** on `demo`/`staging` only. Failures are teaching content; production data never appears on screen.
8. **After each phase** run `/capture-episode`.

## 9. Definition of done (per phase)

- [ ] Acceptance checks in `BUILD_PLAN.md` pass, with evidence
- [ ] `npm run typecheck && npm run lint && npm test` green
- [ ] Runs on `demo` with zero credentials
- [ ] `/brand-audit` clean (no raw hex, contrast pairs approved)
- [ ] Production build shows no TodoChips/StatusChips
- [ ] `DECISIONS.md` / `UNKNOWNS.md` updated
- [ ] Episode notes captured

## 10. Things not to do

- Do not add a second driver, dispatch matching, surge pricing, in-app chat, or a customer-facing driver/ride-tier selector. (A customer vehicle choice is a **flagged future option**, off by default, U-V2.)
- Do not build rider accounts, saved places, or a profile screen. The customer is link-based (see `LAYOUT_AMENDMENTS.md`).
- Do not put the scheduler behind an API call to itself, or into n8n.
- Do not add libraries when 20 lines will do.
- Do not describe legal/insurance/permit status as settled. It is **unverified** and a **launch gate** (`docs/COMPLIANCE_CHECKLIST.md`).
- Do not sound corporate. Direct, clear, technical, honest, demonstrable.
