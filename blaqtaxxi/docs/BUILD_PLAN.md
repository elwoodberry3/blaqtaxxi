# BUILD_PLAN.md — phases, prompts, gates

**Production client build, taught episodically.** Steve pastes the phase prompt into Claude Code. Claude builds, proves it with output, runs `/capture-episode`, then **stops at the gate** until Steve says go. Recording happens on `demo`/`staging` only; production data never appears on camera.

Every phase ends with: acceptance checks shown · `npm run typecheck && npm run lint && npm test` green · `/brand-audit` clean · runs on `demo` with zero credentials · production build shows no chips · `DECISIONS.md` and `UNKNOWNS.md` updated.

**Course arc:** about 16 episodes across 11 phases (0, 0.5, 1–9). "Clip" = a 30–50 s standalone segment.

**Blocking unknowns by phase** (`docs/UNKNOWNS.md`): Phase 0.5 needs SP-1/SP-4 · Phase 3 needs U-L1 · Phase 4 needs U-V1, U-A1 · Phase 5 needs U-D1 (settled by SP-1) · Phase 6 needs U-P1/U-B2 · Phase 9 needs U-G1, U-P2, U-C1–C3.

---

## Episode 0 — Kickoff (no code)

**Show:** the discovery transcript, the client brief, the layout system review, then this scaffold. Why the answers live in files: `CLAUDE.md` (rules), `UNKNOWNS.md` + `DECISIONS.md` (answers and gaps), `SCHEDULER_SPEC.md` (the hard part), `SCREEN_MAP.md` (every screen), skills (how-to), agents (reviewers), MCP (hands).

**Start Claude Code in plan mode, from the repo root:**

```bash
claude --permission-mode plan
```

(Already inside a session? Press `Shift+Tab` until the indicator under the prompt box says plan mode; check the indicator, key behavior can vary by version.) Plan mode can read and research but cannot edit files, which is what this episode needs.

**Then paste this as the first prompt:**

> Read `CLAUDE.md`, then `docs/UNKNOWNS.md`, `docs/DECISIONS.md`, `docs/PRD.md`, `docs/SCREEN_MAP.md`, `docs/SCHEDULER_SPEC.md`, `docs/ARCHITECTURE.md`, `docs/LAYOUT_AMENDMENTS.md`, and `docs/BUILD_PLAN.md`. Do not write code yet. Give me: (1) the mission in two sentences, (2) the three assumptions you consider riskiest, (3) any question that is NOT already answered in DECISIONS.md or listed in UNKNOWNS.md and is hard to reverse (if there are none, say so), (4) which BLOCKING unknowns would stop Phase 0, Phase 0.5, and Phase 1, and (5) your plan for Phase 0.

Claude ends by presenting the Phase 0 plan for approval. **Approving it is the "go":** it exits plan mode and Phase 0 begins. Use `/start-phase <n>` for later phases; drop back into plan mode (`Shift+Tab`) before anything risky.

**Clips:** "Why I write the answers down before I write any code." · "This one sentence defines whether the app is correct." · "I graded the client's design system and it failed contrast. Here's how I know."

---

## Phase 0 — Bootstrap (Ep 1)

> Phase 0. Start from the `ias-build-template` repo (Build 028): clone it into this folder, then apply `build.config.ts` for Build 032. **Re-skin it to the BLAQ tokens**: create `tailwind.config.ts` from the snippet in `docs/BRAND.md` (`blaq-*` colors including the proposed `blaq-gray-text`), remove IAS colors and fonts from the app, and confirm `/brand-audit` is clean. Add `TodoChip` and `StatusChip` gated by `NEXT_PUBLIC_SHOW_TODO_CHIPS` (read inside the function body), plus `APP_ENV` (`demo|staging|production`) and a boot check that makes production fail loudly if a required integration is missing. Set up Vitest, ESLint, and `typecheck`/`test` scripts, `.env.example`, and route groups `(customer)`, `(driver)`, `(admin)` with empty placeholder pages. Confirm `next.config` has no `output: 'export'`. Run it with no credentials and show me, then deploy a Vercel **preview** (not production).

**Acceptance:** `npm run dev` works with zero env vars · a production build renders zero chips (grep the output) · `npm test` runs · preview URL loads · brand audit clean.
**On camera:** the gotcha list in `CLAUDE.md` §7 prevents classic failures before they happen.
**Clip:** "Zero credentials, fully clickable. And why production refuses to boot without them."

---

## Phase 0.5 — Spikes and missing screens (Ep 2)

Two things must happen before serious building: prove the driver-location plan, and draw the screens the layout system does not have.

> Phase 0.5. Load the `realtime-tracking` and `layout-system` skills. **Part A, spikes.** Build a minimal `/spike` page (staging only) that (1) posts the device's location every 15 s to a log table/Redis with a timestamp and visibility state, (2) shows the gap statistics between pings (min, median, max, count over 5-minute windows), and (3) has a button that opens the Google Maps deep link (`https://www.google.com/maps/dir/?api=1&destination=<lat>,<lng>&travelmode=driving&dir_action=navigate`) to a fixed test destination. Write a one-page test protocol (SP-1, SP-4) I can hand the driver: 30 minutes, one run screen-unlocked in Google Maps and one screen-locked, iPhone/Android as applicable, and a template to record the result. Do **not** claim the result; wait for real data. **Part B, screens.** For every ✚ or ◐ screen in `docs/SCREEN_MAP.md`, produce a static wireframe in the layout-system style (same `data-slot` conventions, only `blaq-*` tokens, `blaq-gray-text` for gray text, aria-labels, `dvh`) under `.claude/skills/layout-system/wireframes/`, add each to `manifest.json`, and extend `docs/layout-preview/preview.html` so the client can review every customer, driver, and admin screen in a browser. Run `/brand-audit`.

**Acceptance:** spike page deployed to staging · protocol written · every SCREEN_MAP screen has a wireframe and manifest entry · preview page shows them all · brand audit clean.
**Gate (Steve + client):** the wireframes are approved; the spike results are recorded in `UNKNOWNS.md` (U-D1 resolved: PWA is enough, or a native shell is required for `/drive`).
**On camera:** the spike is the episode: a real drive, real gaps in the data. If it fails, that is the best content in the course.
**Clips:** "Google's turn-by-turn SDK isn't available for web. Here's what that does to my architecture." · "I made the client's app wireframes from a design system that had six screens and needed thirty."

---

## Phase 1 — Scheduler core, test-first (Ep 3–4) ★

> Phase 1. Load the `scheduling-engine` skill. Build `lib/scheduler` exactly per `docs/SCHEDULER_SPEC.md` §3–§5: types, config, `dayWindow` (Luxon, America/Chicago, with per-date overrides), `canInsert`, `validateSchedule`. **Test-first:** write the failing tests for S1, S2, S3, S4, S5a–c, S6a, S7, S8a/b, S10 and S11a–c using the fake provider built from `docs/fixtures/travel-matrix.json`, show them failing, then implement until green. Keep the module pure: no imports from next, db, stripe, fetch, or env. Use the `scheduler-verifier` agent to review before you report done.

**Acceptance:** all listed cases pass with the exact numbers in the spec · purity grep empty · verifier agent reports no invariant violations.
**On camera:** tests fail first, then pass. The prev/next-only invariant. The traffic-blind engine offering 09:20 and the aware engine rejecting it (S2).
**Clips:** "Why a rush-hour ride blocks the ride after it." · "Only two neighbors matter when you insert a ride." · "A drop-off around the corner means the next pickup can be 17 minutes later, not an hour."
**Gate:** Steve reviews S2/S3 output.

---

## Phase 2 — Routing seam, demo traffic model, per-car pricing (Ep 5)

> Phase 2. Load the `scheduling-engine` skill. Implement `lib/routing`: the `TravelTimeProvider` interface, a demo provider reproducing `docs/fixtures/travel-matrix.json` (departure-time peak model; `derivedExamples` must match exactly) plus a haversine fallback, a cache keyed by grid cell × grid cell × hour bucket, and a `google.ts` provider behind `GOOGLE_MAPS_SERVER_KEY` (traffic-aware, future `departureTime`; if unsure of exact request fields, say so and use the docs MCP rather than guessing). Then implement `lib/pricing`: a **`PriceConfig` per car** (ordered tiers on the off-peak reference time, integer cents, versioned) using `docs/fixtures/vehicles.json`, with tests P1–P7. Show provider-call counts for cold vs warm slot search.

**Acceptance:** `derivedExamples` reproduce · P1–P7 pass · UI/API label demo travel times as illustrative · pricing has no hardcoded amounts.
**On camera:** why price ignores traffic but scheduling doesn't; why price is per car and versioned.
**Clip:** "A flat fee is a product decision, and it's why the scheduler owns the traffic risk."

---

## Phase 3 — Slot search, booking flow, customer link (Ep 6–7)

> Phase 3. Load the `scheduling-engine` and `layout-system` skills. Requires U-L1 answered; if it is not, stop and ask. Implement `findSlots` and `earliestRideNow` (spec §7). Build the customer booking flow with the layouts from `SCREEN_MAP.md`: C1 start, C2 slot picker (tight-fit badge, "why is this time missing?"), C3 confirm and price (one flat fare for the assigned car, cancellation terms in plain words, contact fields), C10 link problems. Implement `lib/links`: generate the customer link (`/pickup/<lastname>-<last4>-<random>` unless U-L1 says otherwise), normalize surnames (O'Brien, De La Cruz, hyphenated, non-ASCII), store only a hash, expire after the trip window, rate-limit lookups, and a recovery flow. **Test-first** for links: entropy, collisions, normalization, expiry, brute-force lockout. No payment yet (the C4 step is a flagged stub on `demo`). Add the property tests from spec §9.

**Acceptance:** property tests pass on 200 seeded days · S3 shows 09:25 · link tests pass, including "last name + last four alone never authorizes" · out-of-area shows the contact prompt · brand audit clean.
**On camera:** "Why is 9:00 gone?"; guessing links live against the naive scheme, then the fix.
**Clips:** "Your customers' trip links can be guessed in 10,000 tries. Here's how I fixed it." · "Fixed hourly slots waste his day."

---

## Phase 4 — Data model, admin backend, driver schedule (Ep 8–9)

> Phase 4. Load `demo-mode-seams`, `admin-backend`, and `layout-system`. Requires U-V1 and U-A1 answered. Add the Drizzle + Neon schema per `docs/ARCHITECTURE.md` §3 (including `vehicles`, versioned `price_configs`, `vehicle_assignments`, `users` with roles, `config_audit_log`, `availability_overrides`) with an in-memory fallback on `demo`. Build the **admin backend** at `/admin` (A1–A6, A8, A12, A13 from `SCREEN_MAP.md`): secure sign-in, calendar, availability with per-date overrides and the D-45 conflict list, cars, per-car price configs with live quote preview and versioning, operations settings, audit log. Every admin route authorizes on the server. Build the driver schedule screens D1 and D7 at `/drive`. Wire `POST /api/bookings` with the advisory lock and re-run of `canInsert` inside the transaction, plus the 5-minute hold, snapshotting `priceConfigVersion`. **Tests:** S9 (concurrency), admin authz-denied, "editing a price never changes an existing booking", audit-log append-only.

**Acceptance:** S9 passes · price edit leaves existing bookings unchanged · a non-admin gets 403 on every `/api/admin/*` route · overrides and blocks change slot results immediately · conflicts are listed, not auto-cancelled.
**On camera:** the double-booking bug you'd have without the lock; the "changing a price must not rewrite history" test.
**Clip:** "Two customers tap the same slot at the same time. Here's what actually happens."

---

## Phase 5 — Driver navigation, live tracking, customer trip page (Ep 10–12)

Do not start until the Phase 0.5 spike has settled U-D1.

> Phase 5. Load `realtime-tracking` and `layout-system`. Implement per the U-D1 outcome recorded in `UNKNOWNS.md` (PWA foreground pings, or the native-shell path if the spike failed; if it is unresolved, stop and ask). **Driver** (`/drive`, D2–D6): next pickup with time to spare and late-risk, **Navigate** buttons built by `lib/navigation` (Google Maps deep links, tested), Heading to pickup → Arrived (5-minute wait, no-show enabled after) → **Rider in car** → the primary action becomes **Navigate to drop-off** → Complete, with a location ping on every tap. **Customer** (`/pickup/[link]`, C6–C8): before (details, countdown, "check back 30 minutes before"), approaching (from T-30: Google Map with driver marker, driver card, ETA, "last seen N s ago"), during (car on the route, estimated arrival). Enforce the T-30 gate server-side. Implement `projectDay` late-risk and cascade (L1, L2), driver alerts, and an event for the async layer. Add a "simulate driver" dev tool so all of this is demoable at a desk.

**Acceptance:** V1, L1, L2 pass · API test proves no location leaks before T-30 and no cross-customer leakage · deep-link builder tests pass · stale ping flagged · simulator drives the customer screens · a real-phone run on staging is recorded.
**On camera:** the handoff to Google Maps and what happens to the pings; the honest numbers from the spike.
**Clips:** "Location is only visible 30 minutes out. Here's the API rule." · "How the app knows he's going to be late before he does." · "What happens to live tracking when the driver switches to Google Maps."

---

## Phase 6 — Payments, receipts, and policy (Ep 13) — "Policy to Code"

> Phase 6. Load `demo-mode-seams` and `admin-backend`. Requires U-P1 and U-B2. Implement `lib/payments` with Stripe **test mode** (PaymentIntent at hold, webhook `held → confirmed`, idempotent on event id) and the simulated fallback; build C4 (pay), C5 (booked + link delivery on screen), C9 (trip log and completion receipt with business details from admin), C11 (cancel with the fee shown first) and C12 states. Implement `lib/policy` from versioned policy config (D-52, D-53, D-55) with tests C1–C7, and admin screens A7 (policy), A9 (bookings, cancel/refund), A11 (business profile). Do not enable live mode. If unsure about a Stripe API detail, check current docs via the docs MCP and cite it in a code comment.

**Acceptance:** C1–C7 pass · webhook idempotency test · an unpaid hold expires and frees its slot · receipt shows only real data · policy edits create a new version and never change existing bookings.
**On camera:** a written policy in a table becomes a config file and a test suite, then a business rule vs a legal rule (`COMPLIANCE_CHECKLIST.md`).
**Clip:** "A cancellation policy is just a function. Here's the whole thing."

---

## Phase 7 — n8n async layer and messaging (Ep 14)

> Phase 7. Load `n8n-async-layer` and `admin-backend`. Create workflows under `n8n/workflows/`: booking-confirmed (delivers the customer link), T-30 reminder (schedule trigger → `GET /api/n8n/due` → send → `POST /api/n8n/ack`), late-risk notice, no-show follow-up, receipt, and a daily digest. Use the `Authorization: Bearer` header exactly. Enforce idempotency with `notifications.idempotency_key`. Message copy comes from the admin-editable templates (A10) with a preview. `demo` writes to `/dev/outbox`. Export each workflow as JSON and document credential setup, including project-level sharing vs ownership. SMS stays behind the seam until U-N1 is settled; email works now.

**Acceptance:** duplicate delivery sends once · wrong Bearer returns 401 · outbox shows every message · nothing in the live path awaits n8n · editing a template changes the next message, not past ones.
**On camera:** the credential-sharing silent failure, on purpose.
**Clip:** "Why the ETA screen never touches n8n."

---

## Phase 8 — Harden and verify (Ep 15)

> Phase 8. Run the `pre-deploy-auditor` agent and fix everything it finds. Add Playwright happy paths: customer (book → pay (test) → link → driver completes → receipt) and admin (change availability → slots update). Add Sentry (scrubbing phone/email), PostHog, and Vercel Analytics behind env checks, with alerting to the owner. Add the private optional rating (U-R1). Run an accessibility pass on `/book`, `/pickup/[link]`, `/drive`, `/admin` and fix or list every issue. Load-test slot search against the provider-call budget. Write `docs/CASE_STUDY.md` only with facts that exist in the repo and only after the client agrees to publication (U-C1). Deploy to the **staging** environment with Stripe in test mode.

**Acceptance:** auditor clean · e2e green · a11y issues fixed or listed · case study has no fabricated numbers · alerts fire on a forced error.
**On camera:** the audit, including what it found that you missed.
**Clips:** "The reviewer agent found three things I'd have shipped." · "One build, three doors."

---

## Phase 9 — Launch readiness (Ep 16) — production go-live gate

Nothing here is optional for a paying client. Nothing in this phase ships without written sign-off from Steve **and** the client.

> Phase 9. Produce `docs/LAUNCH_CHECKLIST.md` and work it with me item by item; do not mark anything done without evidence. Cover: production environment on the client-owned Vercel/Neon/Upstash/Google/Stripe accounts (U-C1, U-P2); `www.blaqtaxxi.com` DNS and TLS; production boot check passes with real integrations; production build shows **zero** chips and `/dev/*` routes are unreachable; live Stripe keys loaded **only** in production and a real small charge plus refund tested by the client; SMS registration status (U-N1); terms of service, privacy policy, and location-sharing disclosure in place (U-G2); `COMPLIANCE_CHECKLIST.md` signed off by the responsible people (U-G1); admin 2FA/passkey enabled; backups and restore tested; monitoring, alerts, and budget alerts set; production config seeded through the admin (hours, car, price config, policy); rollback plan; runbook and handoff document; one supervised real-phone rehearsal with the client. Then stop and present the checklist for sign-off. Do not go live yourself.

**Acceptance:** every checklist item has evidence · both sign-offs recorded · rollback tested.
**Gate:** written go-live approval from Steve and the client.

---

## After the build

- **Employment door:** case study + repo + 2-minute walkthrough (with client permission).
- **Consulting door:** extract `lib/scheduler`, `lib/routing`, and the admin config pattern as a starter module; write a one-page offer for solo operators who travel between appointments.
- **Brand door:** assemble the "Build 032" playlist from the episode notes.
