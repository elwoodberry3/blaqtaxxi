@CLAUDE.layout.md

# CLAUDE.md — BLAQTAXXI ("One-to-One Uber")

> Governance file for Claude Code. Read this first, every session. If this file and a chat message disagree, follow the chat message, then tell me so this file can be updated.
> The line above, `@CLAUDE.layout.md`, imports the BLAQ layout/token rules. If your Claude Code version does not expand `@` imports, paste that file's contents under its own heading here.

**This is a production-grade build for a client (the owner-driver of BLAQTAXXI), taught episodically in an advanced Skool course.** When those goals conflict, **the client's product wins**: no client or customer data on camera, no unfinished work visible to customers, no shortcuts that would not survive launch.

**Build:** IAS Build 032 (confirmed) · **Repo:** `elwoodberry3/blaqtaxxi` (IAS-owned during the pilot)
**Governing principle:** One Build. Three Doors. Every phase yields (1) a shippable piece of the client's product, (2) a reusable consulting module, (3) a recordable episode.

---

## 1. How this gets delivered (pilot, then handoff)

1. **Pilot on IAS infrastructure** at `blaqtaxxi.iasbootcamp.com`. IAS owns the domain/DNS, Vercel, Google Cloud, and Stripe accounts during the pilot. The client (the driver) uses it, with written "kick the tires" instructions, for **1–2 feedback rounds** to tune it to his exact needs.
2. **The public may use the pilot too, right up to the moment money would be paid.** No real payment is taken in the pilot. Stripe stays in **test mode**; the checkout step shows a clear pilot notice.
3. **When the client approves the final,** IAS gives him instructions to deploy the same app on **his own** domain, Vercel, Google Cloud, Stripe, and other accounts. Live payments exist only in that client-owned deployment.

So the code must be **deployable by someone else from a guide**: env-driven, seeded through the admin, with a config export/import and an environment doctor script. See `docs/PILOT_PLAYBOOK.md` and `docs/CLIENT_DEPLOY_GUIDE.md`.

## 2. What this product is

One driver. One active fare at a time. The driver owns **several cars** (a black Nissan Sentra today, a luxury SUV as an example) and switches between them by day or shift. **The customer chooses the car when booking**, and each car has its own price configuration. Customers reserve a time, pay a flat fee up front, and follow the driver by link. The core problem is **travel-time-aware scheduling**: a booking is only offered if the driver can reach it on time from wherever his *previous ride ends*, in the traffic at that hour, and still make his *next* pickup.

### Three surfaces (each sees something different)

| Surface | Route | Who / auth | What they do |
|---|---|---|---|
| **Admin backend** | `/admin` | The owner-driver. Secure login | Configure calendar and availability, **cars and when each is assigned**, per-car prices, cancellation policy, service area, message copy, media, and everything that drives what customers see. Export/import config |
| **Driver app** | `/drive`, shipped in a **thin native shell** | Same person; iPhone default, Android supported | Lyft-driver view: schedule, **Open in Google Maps** to the pickup, confirm the customer is in the car, then **Open in Google Maps** to the drop-off. The app does not own the map; the shell keeps sending location in the background |
| **Customer** | `/book` → `/pickup/[link]` | No account. Pays, then receives a link | **Before:** trip details, countdown, "check back 30 minutes before". **During:** the trip on a Google Map with an estimated arrival. **After:** trip log and completion receipt |

The customer link is `/pickup/<lastname>-<last4>-<random>` (e.g. `/pickup/johnson-4821-k9Xp2mQz`), **per trip**. The readable prefix is cosmetic; the random suffix is the secret. **Never authorize on last name + last four alone** (approved, D-85).

## 3. Read order (before writing any code)

1. `docs/UNKNOWNS.md` — what is open and what it blocks. **Do not build on an item marked BLOCKING without asking.**
2. `docs/DECISIONS.md` — answered questions and labeled assumptions. **Do not re-ask anything answered here.**
3. `docs/PRD.md` · 4. `docs/SCREEN_MAP.md` · 5. `docs/SCHEDULER_SPEC.md` · 6. `docs/ARCHITECTURE.md` · 7. `docs/LAYOUT_AMENDMENTS.md` · 8. `docs/BUILD_PLAN.md` · 9. `docs/BRAND.md` · 10. `docs/PILOT_PLAYBOOK.md`
11. `SKILLS.md` — load the relevant skill before working in its area. **Before building any screen, load `layout-system` and read its `manifest.json` first.** Before touching the driver app, load `driver-shell`.

## 4. Non-negotiables (hard constraints)

1. **Baseline Stack + DXP.** Next.js 14 (App Router) · TypeScript (strict) · Tailwind · Vercel · cloud-hosted n8n (`https://iautomateshit.app.n8n.cloud`). Composable, decoupled frontend/backend, modular payment/auth/integration subsystems, shared design tokens. The driver shell is a **thin native wrapper around `/drive`** (approved), not a second product. **iPhone is the default** (TestFlight internal testing), Android is supported, and **IAS never publishes it** (D-121, D-122).
2. **Real-time vs async split.** The live path (pings → ETA → customer screen, booking, slot search) **never touches n8n**. n8n owns only the async layer.
3. **The scheduler is a pure module.** `lib/scheduler/**` has **zero** imports from Next, the DB, Stripe, or any network client. Travel time enters via `TravelTimeProvider`; prices via a `PriceConfig`.
4. **Environments:** `demo` (course/local, fake data, zero credentials), `pilot` (IAS-hosted, real client config, public may use, **Stripe test only**), `production` (the client's own deployment, live payments). Every integration has a demo fallback for `demo` only. `pilot` and `production` **fail loudly at boot** if a required integration is missing. The course is recorded on `demo`.
5. **Chips only in `demo`.** `TodoChip`/`StatusChip` render only when `NEXT_PUBLIC_SHOW_TODO_CHIPS=true` (`demo`). `pilot` shows a `PilotBanner` and a "Send feedback" control instead; it never shows unfinished-work chips to the public. `production` shows neither. Known gaps are tracked in `docs/UNKNOWNS.md` and the pilot's "known limitations" list.
6. **No fabricated numbers.** Demo travel times are an illustrative model, labeled as such. No invented revenue, ride counts, ratings, or testimonials.
7. **Personal data.** No real customer, driver, or client personal data in the repo, fixtures, screenshots, or on camera. **Client images (driver photo, car photos) live in `client-assets/` (gitignored) and are uploaded through the admin**, not committed. In the pilot, contact details entered by the public are discarded when an unpaid hold expires (24 h at most, D-107).
8. **IAS's Stripe is always test mode** (stated). `demo` and `pilot` use test keys only, and the objective is to show the client that it works. No live key loads outside the client's own production deployment, and IAS never takes his real fares (D-119).
9. **Ask before irreversible moves:** live-mode payments, messaging real people, deleting data, force-pushing, paid-API spend beyond the dev tier, changing pilot/production config.
10. **Layout system is the visual baseline.** `blaq-*` tokens only; no raw hex or arbitrary colors. Deviations only as recorded in `docs/LAYOUT_AMENDMENTS.md`, and **where `CLAUDE.layout.md` conflicts with the amendments (for example gray as text), the amendments win.**
11. **Config drives the customer.** Anything a customer sees that the owner might change (hours, cars, prices, policy amounts, copy, media) comes from admin-managed, versioned config, never hardcoded. A price or policy change never rewrites an existing booking.

## 5. Domain rules you must not get wrong

| Rule | Value | Source |
|---|---|---|
| Drivers / cars | **1 driver, N cars**, one ride at a time. Cars are switched by day or shift | stated |
| Car choice | **The customer chooses the car** at booking, from the cars the driver has assigned for that time. One active car → the choice is hidden | stated |
| Price | **Per car**. Default tiers: **$20** ≤30 min, **$25** >30–45, **$30** >45 (off-peak reference trip time). Amounts for other cars come from the admin | stated (tiers confirmed) |
| Service area | Dallas–Fort Worth | stated |
| Availability | default **05:00–24:00** America/Chicago; override any date | stated |
| Cutoff | the driver's end time is the last time a ride can be **booked to start**; a ride may finish after it | stated (reading to confirm, D-09) |
| Car assignment | Cars are assigned **by day or shift**, with no switching within a shift. A car is offered only inside its windows; a change of car creates a **swap block** at the car base (default 30 min) | stated (D-84) |
| Cancellation | free until T-60 min; $5 inside 60 min; $10 once the driver is en route or on a no-show; config per policy version | Lyft 60-min rule verified; amounts assumed (D-52) |
| Customer visibility | countdown always; **live location + ETA only from T-30 min** | stated |
| Payment | book → pay → flat fee, price locked at booking. **Not taken in the pilot** | stated |
| Driver navigation | Google Maps app via deep link; on "Rider in car" the app offers the route to the drop-off | stated |
| Pricing vs traffic | rush hour affects **scheduling**, never **price** | assumed (D-07) |

**The one sentence that defines correctness:** *A booking is valid only if `prevRide.end + travel(prev.dropoff → new.pickup, at that time) + buffer ≤ new.pickup` AND `new.end + travel(new.dropoff → next.pickup, at that time) + buffer ≤ next.pickup`, the pickup is no later than that date's cutoff, and the chosen car is assigned at that time.* Rush hour makes `travel(...)` bigger; a drop-off "around the corner" makes it tiny, so the next pickup can be minutes later, not an hour later.

## 6. Repo layout (target)

```
app/
  (customer)/book/           route+date → car → time → confirm → pay
  (customer)/pickup/[link]/  before · approaching · in trip · after
  (driver)/drive/            schedule, Open in Google Maps, confirm rider, status flow
  (admin)/admin/             calendar, cars+assignments, prices, policy, area, copy, media, bookings, config export/import
  api/
lib/  scheduler/ (PURE) routing/ pricing/ policy/ tracking/ navigation/ links/ payments/ notify/ db/ auth/ config/ media/
components/                  ui/ (layout-system), PilotBanner, TodoChip, StatusChip
native/driver-shell/         thin Capacitor-style shell for /drive (background location, deep links)
scripts/                     doctor (env + integration check), seed, config export/import
public/brand/                wordmark, favicon set (from client-assets/)
client-assets/               GITIGNORED client images; uploaded via admin, never committed
n8n/workflows/  tests/  docs/  .claude/
```

## 7. Coding conventions

- TypeScript `strict`. No `any` in `lib/scheduler`. Discriminated unions for states.
- Times stored **UTC**, displayed **America/Chicago**, via one `lib/time.ts`. DST-day tests.
- Money is **integer cents**.
- **Styling: `blaq-*` tokens only** (`navy`, `royal`, `green`, `red`, `amber`, `gray`, `gray-text`, `black`, `canvas`, `white`). Red is destructive/negative only; **amber is the warning/late-risk color** (approved). Contrast pairs in `docs/BRAND.md`.
- **Type:** Momo Trust Display (Google Fonts, confirmed by the client) for the wordmark and headings; system font stack for UI text unless the client says otherwise. **Self-host it** with `next/font/local` from files committed to the repo (avoids the build-time Google fetch failure and third-party requests). Confirm the font's license and available weights when you download it.
- Accessibility: WCAG AA, real `<button>`s, focus rings, `aria-live` for ETA, 44 px targets on `/drive` and `/admin`.
- Zod on every route handler; typed errors; admin routes check role server-side.
- Small commits, one concern each, conventional messages.

## 8. Known gotchas

1. **`output: 'export'` silently kills API routes.** Never set it in the Next.js app.
2. **`NEXT_PUBLIC_*` vars are inlined at build time.** Read them inside the function/component body, never through a module-level `const` chain.
3. **`next/font/google` fails at build** when `fonts.googleapis.com` is blocked. Use `next/font/local` with committed font files.
4. **n8n auth:** header key exactly `Authorization`, value includes `Bearer `. Credential *ownership* vs project *sharing* causes silent 401s.
5. **Version drift:** never cherry-pick files from different patch drops.
6. **HubSpot Free ceiling:** 10 custom contact properties.
7. **Vercel functions are not long-lived sockets.** Poll the customer page (10 s in window); short POSTs for pings. No WebSocket/SSE for the MVP.
8. **Background location needs the native shell.** *Verified:* Google's Navigation SDK is native-only (no web) and Maps URLs launch turn-by-turn with no key. Because the driver leaves our app for Google Maps, a web page would be backgrounded, so the shell must keep sending location (Android foreground service; iOS background location). **Do not assume it works: spike SP-1 on a real iPhone (the default platform) and a real Android phone before Phase 5. A physical iPhone and a Mac are needed (U-D7), and simulators are not evidence (my understanding, unverified).** Store distribution rules and plugin choices are unverified (`driver-shell` skill).
9. **Routing APIs cost money per element.** Cache by (origin cell, destination cell, hour bucket).
10. **Google Maps keys:** browser key (Maps JS, referrer-restricted) and server key (Routes, restricted). Never ship the server key to the client. Pricing not verified.
11. **The customer link is a bearer secret** (approved format). At least 64 random bits, hashed at rest, rate-limited, expiring, never logged.
12. **Never leak across customers.**
13. **Price and policy changes are versioned;** a booking snapshots them.
14. **The driver may switch phones (iPhone is the default target; his phone is Android today).** Pings carry a `device_id`; only the device signed in for the active shift is accepted; a new device sign-in requires re-auth and revokes the old device's session.
15. **The wordmark PNG is 195×75 and opaque** (white background baked in, verified). It cannot sit on the navy header and looks soft on high-density screens. Use the reversed/transparent/SVG version once supplied (U-B4); until then, place it only on white.
16. **Car photos supplied are manufacturer-style stock images.** Replace with the driver's real cars before the client's launch (U-B5); the Suburban is labeled an example.

17. **Client images never go into git (D-125).** `client-assets/` is gitignored and a pre-commit guard blocks it; verify with `git ls-files` before every push. A public repo plus a real person's photo is a privacy incident.
18. **Pin Next.js to the latest 14.2.x patch and never rely on middleware alone for authorization** (D-124).

## 9. Working agreement

1. **Repo-first.** Read the actual current files before changing anything.
2. **Structured decisions.** Anything architectural or irreversible not in `DECISIONS.md` → stop and ask with `AskUserQuestion` (2–4 options, recommended first). Cheap-to-reverse → decide, log as `assumed`, continue. **BLOCKING unknowns are never assumed.**
3. **Phase gates.** Follow `docs/BUILD_PLAN.md`; do not start Phase N+1 until N's checks pass and I say go.
4. **Tests first** for `lib/scheduler`, `lib/pricing`, `lib/policy`, `lib/links`, `lib/navigation`.
5. **Show, don't claim.** Run it and show output.
6. **Screens:** load `layout-system`, read `manifest.json`, match the nearest layout; if none fits, build from `components/` and tokens, then add it to `SCREEN_MAP.md`.
7. **Narrate for the camera** on `demo` only. Failures are teaching content; client and customer data never appear on screen.
8. **After each phase** run `/capture-episode`.

## 10. Definition of done (per phase)

- [ ] Acceptance checks in `BUILD_PLAN.md` pass, with evidence
- [ ] `npm run typecheck && npm run lint && npm test` green
- [ ] Runs on `demo` with zero credentials
- [ ] `/brand-audit` clean
- [ ] `pilot` and `production` builds render no chips; `pilot` shows the PilotBanner
- [ ] `DECISIONS.md` / `UNKNOWNS.md` updated
- [ ] Episode notes captured (demo material only)

## 11. Things not to do

- No second driver, dispatch matching, surge pricing, or in-app chat.
- No customer accounts, saved places, or profile screens.
- No real payment processing through IAS's accounts (U-P2).
- No mid-day car swaps modeled beyond the swap block, no per-car scheduling (the resource is the driver).
- Do not put the scheduler behind an API call to itself, or into n8n.
- Do not add libraries when 20 lines will do.
- Do not describe legal/insurance/permit status as settled (`docs/COMPLIANCE_CHECKLIST.md`).
- Do not sound corporate. Direct, clear, technical, honest, demonstrable.
