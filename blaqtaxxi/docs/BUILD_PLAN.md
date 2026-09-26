# BUILD_PLAN.md — phases, prompts, gates

**Production-grade client build, delivered as a pilot and then a handoff, and taught episodically.** Steve pastes the phase prompt into Claude Code. Claude builds, proves it with output, runs `/capture-episode`, then **stops at the gate** until Steve says go. Recording happens on `demo` only; client and customer data never appear on camera.

Every phase ends with: acceptance checks shown · `npm run typecheck && npm run lint && npm test` green · `/brand-audit` clean · runs on `demo` with zero credentials · `pilot`/`production` builds show no chips · `DECISIONS.md` and `UNKNOWNS.md` updated.

**Course arc:** about 17 episodes across 12 phases (0, 0.5, 1–10). "Clip" = a 30–50 s standalone segment.

**Blocking unknowns by phase** (`docs/UNKNOWNS.md`): Phase 0.5 needs U-D7 (a physical iPhone and a Mac) · Phase 4 needs U-A1 · Phase 5 needs SP-1/SP-2 results · Phase 6 needs U-P1, U-B2 · Phase 9 needs U-G1/U-G2 (public pilot notice), U-F1 · Phase 10 needs U-H1, U-G1, and the client's own store accounts (U-D4).

**Human steps Claude cannot do from a cloud session:** build and install the iOS app (needs a Mac, Xcode, and an Apple developer account), run the real-phone drive tests, register carrier/SMS accounts, create Google Cloud and Stripe accounts. Claude prepares code and exact instructions for these and stops.

---

## Episode 0 — Kickoff (no code)

**Show:** the discovery transcript, the client brief, the layout system review, then this scaffold. Why the answers live in files: `CLAUDE.md` (rules), `UNKNOWNS.md` + `DECISIONS.md` (answers and gaps), `SCHEDULER_SPEC.md` (the hard part), `SCREEN_MAP.md` (every screen), skills (how-to), agents (reviewers), MCP (hands).

**Start Claude Code in plan mode, from the repo root:**

```bash
claude --permission-mode plan
```

(Already inside a session? Press `Shift+Tab` until the indicator under the prompt box says plan mode; check the indicator, key behavior can vary by version.) Plan mode can read and research but cannot edit files.

**Then paste this as the first prompt:**

> Read `CLAUDE.md`, then `docs/UNKNOWNS.md`, `docs/DECISIONS.md`, `docs/PRD.md`, `docs/SCREEN_MAP.md`, `docs/SCHEDULER_SPEC.md`, `docs/ARCHITECTURE.md`, `docs/LAYOUT_AMENDMENTS.md`, `docs/PILOT_PLAYBOOK.md`, and `docs/BUILD_PLAN.md`. Do not write code yet. Give me: (1) the mission in two sentences, (2) the three assumptions you consider riskiest, (3) any question that is NOT already answered in DECISIONS.md or listed in UNKNOWNS.md and is hard to reverse (if there are none, say so), (4) which BLOCKING unknowns would stop Phase 0, Phase 0.5, and Phase 1, and (5) your plan for Phase 0.

Claude ends by presenting the Phase 0 plan for approval. **Approving it is the "go":** it exits plan mode and Phase 0 begins. Use `/start-phase <n>` for later phases; drop back into plan mode (`Shift+Tab`) before anything risky.

**Clips:** "Why I write the answers down before I write any code." · "I graded the client's design system and it failed contrast. Here's how I know."

---

## Phase 0 — Bootstrap (Ep 1)

> Phase 0. **Repo hygiene first.** Confirm the scaffold sits at the repo root (not in a nested folder), that `.gitignore` ignores `client-assets/`, that `git ls-files` shows no client image, and install the guard with `git config core.hooksPath .githooks` (tell me before you change git config). Then create the app with a **fresh `create-next-app@14`** (App Router, TypeScript strict, Tailwind, ESLint, npm), pin the latest 14.2.x patch, and hand-write `build.config.ts` for Build 032 (the `ias-build-template` repo holds only a README, D-04). **Style with the BLAQ tokens:** create `tailwind.config.ts` from `docs/BRAND.md` (`blaq-*` including `blaq-gray-text` and `blaq-amber`), remove IAS colors and fonts from the app, and confirm `/brand-audit` is clean. Copy `client-assets/wordmark__blaq.png` and `favicon.jpg` into `public/brand/`, generate the favicon sizes, and place the wordmark **only on white** (it is opaque; see `UNKNOWNS.md` §3). Self-host **Momo Trust Display** with `next/font/local`: if you cannot download the font files in this environment, stop and tell me exactly which files to add and confirm its license and available weights. Add `.gitignore` for `client-assets/`. Add `TodoChip`/`StatusChip` (gated by `NEXT_PUBLIC_SHOW_TODO_CHIPS`, `demo` only), a `PilotBanner` (gated by `NEXT_PUBLIC_PILOT_BANNER`), and `APP_ENV` (`demo|pilot|production`) with a boot check that makes `pilot` and `production` fail loudly if a required integration is missing. Set up Vitest, ESLint, `typecheck`/`test` scripts, `.env.example`, and route groups `(customer)`, `(driver)`, `(admin)` with placeholder pages. Confirm `next.config` has no `output: 'export'`. Run it with no credentials and show me, then deploy a Vercel **preview** to the IAS project.

**Acceptance:** `npm run dev` works with zero env vars · `pilot`/`production` builds render zero chips (grep the output) and only `pilot` renders the banner · `npm test` runs · preview loads · brand audit clean · fonts load without a Google request.
**On camera:** the gotcha list in `CLAUDE.md` §8 prevents classic failures before they happen.
**Clip:** "Zero credentials, fully clickable. And why the pilot and production refuse to boot without them."

---

## Phase 0.5 — Spikes and missing screens (Ep 2–3)

Two things must happen before serious building: prove the driver-location plan on real phones, and draw the screens the layout system does not have.

> Phase 0.5. Load `driver-shell`, `realtime-tracking`, and `layout-system`. **Part A, spikes.** Scaffold `native/driver-shell/` (a thin Capacitor-style shell that loads a hosted URL) plus a minimal `/spike` page (`demo`/`pilot` only). The shell must: request background location, post a ping every 15 s with a `device_id` and timestamp, show ping-gap statistics (min, median, max), and have a button that opens the Google Maps deep link (`https://www.google.com/maps/dir/?api=1&destination=<lat>,<lng>&travelmode=driving&dir_action=navigate`). Write step-by-step instructions for me to build and install it on **an iPhone (the default platform, through TestFlight internal testing on IAS's Apple Developer account) and an Android phone (Play internal testing, assumed)** (I will run Xcode/Android tooling), and a test protocol for SP-1, SP-2, and SP-4: 30 minutes driving with Google Maps in front, once screen unlocked and once locked, on each platform, with a results template (gaps, battery, prompts). Evaluate at least two background-location plugin candidates, say what you could and could not verify, and do **not** claim results. **Part B, screens.** For every ✚ or ◐ screen in `docs/SCREEN_MAP.md` (including the car choice step C2a, the pilot banner and feedback control, and the driver shell states D8–D9), produce a static wireframe in the layout-system style (same `data-slot` conventions, only `blaq-*` tokens, `blaq-gray-text` for gray text, aria-labels, `dvh`), add each to `manifest.json`, and extend `docs/layout-preview/preview.html` so the client can review every customer, driver, and admin screen. Run `/brand-audit`.

**Acceptance:** shell and spike page build instructions written and reviewed by me · protocol written · every SCREEN_MAP screen has a wireframe and manifest entry · preview shows them all · brand audit clean.
**Gate (Steve + client):** wireframes approved; **SP-1, SP-2, SP-4 results recorded in `UNKNOWNS.md`** and U-D6 settled.
**On camera:** the spike is the episode: a real drive, real gaps in the data. If it fails, that is the best content in the course.
**Clips:** "Google's turn-by-turn SDK isn't available for web. Here's what that does to my architecture." · "The design system had six screens and the product needed thirty."

---

## Phase 1 — Scheduler core, test-first (Ep 4–5) ★

> Phase 1. Load the `scheduling-engine` skill. Build `lib/scheduler` exactly per `docs/SCHEDULER_SPEC.md` §3–§5: types, config, `dayWindow` (Luxon, America/Chicago, with per-date overrides), `canInsert` (including the car-window check and `CAR_NOT_ASSIGNED`), `validateSchedule`. **Test-first:** write the failing tests for S1, S2, S3, S4, S5a–c, S6a, S7, S8a/b, S10, S11a–c, and S12a–c using the fake provider from `docs/fixtures/travel-matrix.json` and the car windows in `docs/fixtures/vehicles.json`, show them failing, then implement until green. Keep the module pure: no imports from next, db, stripe, fetch, or env. Use the `scheduler-verifier` agent before you report done.

**Acceptance:** all listed cases pass with the exact numbers in the spec · purity grep empty · verifier reports no invariant violations.
**On camera:** tests fail first, then pass. The prev/next-only invariant. The traffic-blind engine offering 09:20 and the aware engine rejecting it (S2). The swap block that stops him being stranded (S12c).
**Clips:** "Why a rush-hour ride blocks the ride after it." · "Only two neighbors matter when you insert a ride." · "Switching cars is just a block on the calendar."
**Gate:** Steve reviews S2/S3/S12 output.

---

## Phase 2 — Routing seam, demo traffic model, per-car pricing (Ep 6)

> Phase 2. Load the `scheduling-engine` skill. Implement `lib/routing`: the `TravelTimeProvider` interface, a demo provider reproducing `docs/fixtures/travel-matrix.json` (departure-time peak model; `derivedExamples` must match exactly) plus a haversine fallback, a cache keyed by grid cell × grid cell × hour bucket, and a `google.ts` provider behind `GOOGLE_MAPS_SERVER_KEY` (traffic-aware, future `departureTime`; if unsure of exact request fields, say so and use the docs MCP rather than guessing). Then implement `lib/pricing`: a **`PriceConfig` per car** (ordered tiers on the off-peak reference time, integer cents, versioned) using `docs/fixtures/vehicles.json`, with tests P1–P7. Show provider-call counts for cold vs warm slot search, including a two-car search.

**Acceptance:** `derivedExamples` reproduce · P1–P7 pass · demo travel times labeled illustrative · no hardcoded amounts.
**On camera:** why price ignores traffic but scheduling doesn't; why price is per car and versioned.
**Clip:** "A flat fee is a product decision, and it's why the scheduler owns the traffic risk."

---

## Phase 3 — Slot search, car choice, booking flow, customer link (Ep 7–8)

> Phase 3. Load the `scheduling-engine` and `layout-system` skills. Implement `findSlots` (per car) and `earliestRideNow` (spec §7). Build the customer booking flow from `SCREEN_MAP.md`: C1 start; **C2a choose a car** (only cars with a slot and enough seats, each with photo, model, seats, and its price for this trip; hidden when one car); C2 pick a time for that car (tight-fit badge, "why is this time missing?" including the car not being assigned); C3 confirm and price; C10 link problems. Implement `lib/links` for the **approved** link `/pickup/<lastname>-<last4>-<random>`: normalize surnames (O'Brien, De La Cruz, hyphenated, non-ASCII), ≥64 random bits, hash at rest, expire, rate-limit, recover. **Test-first** for links: entropy, collisions, normalization, expiry, brute-force lockout, "last name + last four alone never authorizes." C4 is a stub on `demo`. Add the property tests from spec §9.

**Acceptance:** property tests pass on 200 seeded days · S3 shows 09:25 · S12a–d behave in the UI · link suite passes · out-of-area shows the contact prompt · brand audit clean.
**On camera:** "Why is 9:00 gone?"; guessing links live against the naive scheme, then the fix.
**Clips:** "Your customers' trip links can be guessed in 10,000 tries. Here's how I fixed it." · "The customer picks the car; the scheduler decides if it can be there."

---

## Phase 4 — Data model, admin backend, driver schedule (Ep 9–10)

> Phase 4. Load `demo-mode-seams`, `admin-backend`, and `layout-system`. Requires U-A1 answered (mid-day switching is settled: none, D-84). Add the Drizzle + Neon schema per `docs/ARCHITECTURE.md` §4 (`vehicles`, `vehicle_assignments`, versioned `price_configs`, `policies`, `media`, `driver_devices`, `users` with roles, `config_audit_log`, `availability_overrides`) with an in-memory fallback on `demo`. Build the **admin backend** (A1–A6, A8, A12–A14): secure sign-in, calendar, availability with per-date overrides and the D-45 conflict list, **cars with photo upload, assignment calendar (no overlaps), and "Switch car" creating a 30-minute swap block**, per-car price configs with live quote preview and versioning, operations settings, audit log, **config export/import**. Every admin route authorizes on the server. Build the driver schedule screens D1 and D7 and the device registry (D9). Wire `POST /api/bookings` with the advisory lock, re-run of `canInsert`, the 5-minute hold, and the snapshot of `priceConfigVersion`. **Tests:** S9, S12a–c through the API, admin authz-denied, "editing a price never changes an existing booking", audit-log append-only, config export excludes bookings and customer data, export→import round-trip.

**Acceptance:** S9 passes · price edit leaves existing bookings unchanged · a non-admin gets 403 on every `/api/admin/*` route · overlapping car windows are rejected · round-trip config import reproduces the same slots · conflicts are listed, not auto-cancelled.
**On camera:** the double-booking bug you'd have without the lock; "changing a price must not rewrite history."
**Clip:** "Two customers tap the same slot at the same time. Here's what actually happens."

---

## Phase 5 — Driver shell, navigation, live tracking, customer trip page (Ep 11–13)

Do not start until the Phase 0.5 spike results are recorded and U-D6 is settled.

> Phase 5. Load `driver-shell`, `realtime-tracking`, and `layout-system`. Build the shell per the recorded spike result (iPhone default, Android supported; hosted `/drive` unless U-D6 says otherwise): background location with `device_id`, permission and battery states (D8), new-device sign-in (D9), deep links via `lib/navigation`. **Driver** (D2–D6): next pickup with time to spare and late-risk, **Open in Google Maps** buttons, Heading to pickup → Arrived (5-minute wait, no-show enabled after) → **Rider in car** → the primary action becomes **Open in Google Maps** to the drop-off → Complete, with a ping on every tap. **Customer** (`/pickup/[link]`, C6–C8): before (details, chosen car, countdown, "check back 30 minutes before"), approaching (from T-30: Google Map with driver marker, driver card with the assigned car, ETA, "last seen N s ago"), during (car on the route, estimated arrival). Enforce the T-30 gate server-side. Implement `projectDay` late-risk and cascade (L1, L2) with the amber warning treatment, and an event for the async layer. Add a "simulate driver" dev tool. Reject pings from revoked devices.

**Acceptance:** V1, L1, L2 pass · API test proves no location leaks before T-30 and no cross-customer leakage · revoked-device pings rejected · deep-link tests pass · stale ping flagged · simulator drives the customer screens · **a recorded real-phone run on the iPhone (default) and on Android**; if no physical iPhone is available (U-D7), iOS is labeled unverified.
**On camera:** the handoff to Google Maps and what the pings do; the honest numbers from the spike.
**Clips:** "Location is only visible 30 minutes out. Here's the API rule." · "How the app knows he's going to be late before he does." · "The driver app doesn't own the map, and that's on purpose."

---

## Phase 6 — Payments, receipts, and policy (Ep 14) — "Policy to Code"

> Phase 6. Load `demo-mode-seams` and `admin-backend`. Requires U-P1 and U-B2. Implement `lib/payments` with Stripe **test mode** (PaymentIntent at hold, webhook `held → confirmed`, idempotent on event id) and the simulated fallback. **Pilot behavior (D-108):** only allow-listed testers can complete a test payment; everyone else sees the pilot notice at C4 and the hold expires (contact data purged within 24 h, D-107). Build C4, C5 (booked + link on screen), C9 (trip log and completion receipt with business details from admin), C11 (cancel with the fee shown first), C12. Implement `lib/policy` from versioned policy config (D-52, D-53, D-55) with tests C1–C7, and admin screens A7, A9, A11. **IAS's Stripe is always test mode (D-119): never process real payments.** If unsure about a Stripe API detail, check current docs via the docs MCP and cite it in a code comment.

**Acceptance:** C1–C7 pass · webhook idempotency test · unpaid hold expires, frees its slot, and purges contact data · a non-allow-listed user cannot complete a test payment · receipt shows only real data · policy edits create a new version and never change existing bookings.
**On camera:** a written policy in a table becomes a config file and a test suite; business rule vs legal rule.
**Clip:** "A cancellation policy is just a function. Here's the whole thing."

---

## Phase 7 — n8n async layer, messaging, pilot feedback (Ep 15)

> Phase 7. Load `n8n-async-layer` and `admin-backend`. Create workflows under `n8n/workflows/`: booking-confirmed (delivers the customer link), T-30 reminder (schedule trigger → `GET /api/n8n/due` → send → `POST /api/n8n/ack`), late-risk notice, no-show follow-up, receipt, **pilot feedback** (from `POST /api/feedback` to a shared sheet/Slack), and a daily digest. Use the `Authorization: Bearer` header exactly. Enforce idempotency with `notifications.idempotency_key`. Message copy comes from the admin-editable templates (A10) with a preview. `demo` writes to `/dev/outbox`. **In the pilot no SMS goes to the public** and email goes only to allow-listed testers (U-N1). Export each workflow as JSON and document credential setup, including project-level sharing vs ownership.

**Acceptance:** duplicate delivery sends once · wrong Bearer returns 401 · outbox shows every message · nothing in the live path awaits n8n · editing a template changes the next message, not past ones · pilot sends nothing to non-allow-listed recipients.
**On camera:** the credential-sharing silent failure, on purpose.
**Clip:** "Why the ETA screen never touches n8n."

---

## Phase 8 — Harden and verify (Ep 16)

> Phase 8. Run the `pre-deploy-auditor` agent and fix everything it finds. Add Playwright happy paths: customer (choose car → book → test pay as an allow-listed tester → link → driver completes → receipt) and admin (change availability and switch a car → slots update). Add Sentry (scrubbing phone/email), PostHog, and Vercel Analytics behind env checks, with alerting to the owner. Add the private optional rating (U-R1). Run an accessibility pass on `/book`, `/pickup/[link]`, `/drive`, `/admin` and fix or list every issue. Load-test slot search against the provider-call budget with two cars. Write `docs/CASE_STUDY.md` only with facts that exist in the repo and only after the client agrees to publication (U-C1).

**Acceptance:** auditor clean · e2e green · a11y issues fixed or listed · no fabricated numbers · alerts fire on a forced error.
**On camera:** the audit, including what it found that you missed.
**Clips:** "The reviewer agent found three things I'd have shipped." · "One build, three doors."

---

## Phase 9 — Pilot on IAS infrastructure (Ep 17)

The client kicks the tires; the public may use it up to the payment step.

> Phase 9. Load `demo-mode-seams`. Deploy the app to `blaqtaxxi.iasbootcamp.com` on IAS's Vercel and Google Cloud projects with `APP_ENV=pilot`, Stripe **test** keys, the `PilotBanner`, the "Send feedback" control, the pilot payment notice, and a "Known limitations" page generated from `docs/UNKNOWNS.md`. Seed the pilot from `docs/fixtures/vehicles.json` and the client's config via the admin (photos uploaded from `client-assets/`; the Suburban is labeled **example** until U-V5 is answered). Add the pilot privacy notice and "pilot, no rides are booked" terms as drafts for counsel (U-G1, U-G2). Complete `docs/PILOT_PLAYBOOK.md`: the driver's how-to-use guide, the kick-the-tires checklist by feature, the feedback rounds, and the exit criteria. Run `scripts/doctor` against the pilot and show me. Then stop: the human rounds (client testing, feedback triage, tuning) follow the playbook. After each round, apply config changes through the admin and code changes through phases, and record the client's written approval.

**Acceptance:** doctor clean · pilot renders the banner and no chips · a non-allow-listed visitor reaches the payment step and stops, with no ride booked and data purged within 24 h · the client completes the full loop with test payments · feedback flows to the sheet · playbook complete.
**Gate:** written approval of the final from the client (Steve records it).

---

## Phase 10 — Handoff and the client's own go-live (Ep 18)

Nothing here ships without written sign-off from Steve **and** the client.

> Phase 10. Produce `docs/CLIENT_DEPLOY_GUIDE.md` (verified step by step on a clean set of accounts: domain and DNS, Vercel, Neon, Upstash, Google Cloud keys with restrictions, Stripe live account and webhook, Resend, SMS status, n8n or its replacement, object storage, environment variables, migrations, seed, admin bootstrap and 2FA/passkey, config import from the pilot, the native driver app under his own Apple/Google accounts (U-D4), and rollback). Finish `scripts/doctor`, the seed and migration scripts, and a config export/import round-trip test. Produce `docs/LAUNCH_CHECKLIST.md` and work it with me item by item; do not mark anything done without evidence: production boot check with real integrations; **zero** chips and no pilot banner in the production build; `/dev/*` and dev sign-in unreachable; live Stripe keys only in his production; a real small charge and refund tested by him; SMS registration status; terms, privacy policy, and location disclosure in place; `COMPLIANCE_CHECKLIST.md` signed off; admin 2FA/passkey; backups and a tested restore; monitoring and budget alerts; a supervised real-phone rehearsal on Android and iPhone; runbook and support agreement (U-H1). Then stop and present the checklist for sign-off. Do not go live yourself.

**Acceptance:** guide followed end to end by someone other than the author · every checklist item has evidence · both sign-offs recorded · rollback tested.
**Gate:** written go-live approval from Steve and the client.

---

## After the build

- **Employment door:** case study + repo + 2-minute walkthrough (with client permission).
- **Consulting door:** extract `lib/scheduler`, `lib/routing`, the admin-config pattern, and the pilot-to-handoff playbook as a starter kit for solo operators who travel between appointments.
- **Brand door:** assemble the "Build 032" playlist from the episode notes.
