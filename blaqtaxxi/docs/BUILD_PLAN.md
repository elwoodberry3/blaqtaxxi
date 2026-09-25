# BUILD_PLAN.md — phases, prompts, gates

How to use: Steve pastes the phase prompt into Claude Code. Claude builds, proves it with output, runs `/capture-episode`, then **stops at the gate** until Steve says go.

Every phase must end with: acceptance checks shown · `npm run typecheck && npm run lint && npm test` green · runs in demo mode with zero credentials · new gaps visible as `TodoChip`s · `DECISIONS.md` updated.

Suggested course arc: **13 episodes** across 9 phases. "Clip" = a 30–50 s standalone segment for short-form distribution.

---

## Episode 0 — Kickoff (no code)

**Show:** the discovery transcript, then this scaffold. Explain why the answers live in files: `CLAUDE.md` (rules), `DECISIONS.md` (answers), `SCHEDULER_SPEC.md` (the hard part), skills (how-to), agents (reviewers), MCP (hands).

**First prompt to Claude Code:**

> Read `CLAUDE.md`, then `docs/DECISIONS.md`, `docs/PRD.md`, `docs/SCHEDULER_SPEC.md`, `docs/ARCHITECTURE.md`, and `docs/BUILD_PLAN.md`. Do not write code yet. Give me: (1) the mission in two sentences, (2) the three assumptions you consider riskiest, (3) any question that is NOT already answered in DECISIONS.md and is hard to reverse. If there are none, say so and wait for "go" to start Phase 0.

**Clip ideas:** "Why I write the answers down before I write any code." · "This one sentence defines whether the app is correct."

---

## Phase 0 — Bootstrap (Ep 1)

> Phase 0. Start from the `ias-build-template` repo (Build 028): clone it into this folder, then apply `build.config.ts` for Build 032 (values in `docs/DECISIONS.md` D-01…D-04). Wire Tailwind to the tokens in `docs/BRAND.md` via CSS variables, add `TodoChip` and `StatusChip`, set up Vitest, ESLint, and `typecheck`/`test` scripts, add `.env.example`, and make the home page show StatusChips for every phase (all `not started` except Phase 0). Confirm `next.config` has no `output: 'export'`. Fonts via runtime `<link>`. Run it with no credentials and show me. Then deploy a Vercel preview.

**Acceptance:** `npm run dev` works with zero env vars · home page shows phase chips · `npm test` runs (even if empty) · preview URL loads.
**On camera:** the gotcha list in `CLAUDE.md` §7 prevents three classic failures before they happen.
**Clip:** "Zero credentials, fully clickable. Here's the seam."

---

## Phase 1 — Scheduler core, test-first (Ep 2–3) ★ the star

> Phase 1. Load the `scheduling-engine` skill. Build `lib/scheduler` exactly per `docs/SCHEDULER_SPEC.md` §3–§5: types, config, `dayWindow` (Luxon, America/Chicago), `canInsert`, `validateSchedule`. **Test-first:** write the failing tests for S1, S2, S3, S4, S5a/b, S6a, S7, S8a/b and S10 using the fake provider built from `docs/fixtures/travel-matrix.json`, show them failing, then implement until green. Keep the module pure: no imports from next, db, stripe, fetch, or env. Use the `scheduler-verifier` agent to review before you report done.

**Acceptance:** all listed cases pass with the exact numbers in the spec · `grep` shows no forbidden imports in `lib/scheduler` · verifier agent reports no invariant violations.
**On camera:** watch tests fail first, then pass. Explain the prev/next-only invariant. Show the traffic-blind engine offering 09:20 and the aware engine rejecting it (S2).
**Clips:** "Why a rush-hour ride blocks the ride after it." · "Only two neighbors matter when you insert a ride — here's why." · "A drop-off around the corner means the next pickup can be 17 minutes later, not an hour."

**Gate:** Steve reviews S2/S3 output before Phase 2.

---

## Phase 2 — Routing seam, demo traffic model, pricing (Ep 4)

> Phase 2. Load the `scheduling-engine` skill. Implement `lib/routing`: the `TravelTimeProvider` interface, a demo provider that reproduces `docs/fixtures/travel-matrix.json` (departure-time based peak model, `derivedExamples` must match exactly) plus a haversine fallback for arbitrary points, a cache keyed by grid cell × grid cell × hour bucket, and a `google.ts` provider behind `GOOGLE_MAPS_API_KEY` (traffic-aware, future `departureTime`; if you are unsure of the exact request fields, say so and add a TodoChip rather than guessing). Then implement `lib/pricing` per D-07 with tests P1–P5. Show the provider-call count for a cold vs warm slot search.

**Acceptance:** `derivedExamples` all reproduce · P1–P5 pass · UI/API responses label demo travel times as illustrative.
**On camera:** why price ignores traffic but scheduling doesn't. Cost control by caching.
**Clip:** "A flat fee is a product decision, and it's why the scheduler owns the traffic risk."

---

## Phase 3 — Slot search + rider booking UI (Ep 5–6)

> Phase 3. Load the `scheduling-engine` skill. Implement `findSlots` and `earliestRideNow` (spec §7) with the provider call budget, then build the rider flow at `/book`: pickup and drop-off (fixed fake DFW list in demo mode), date, party size → slot list with a "tight fit" badge and a "why is this time missing?" tooltip using the suppressed-reason data → locked quote → contact form. No payment yet: render a TodoChip. Add the property tests from spec §9 (soundness, completeness on the grid, monotone insertion, determinism, provider budget). Mobile-first, WCAG AA.

**Acceptance:** property tests pass on 200 seeded random days · slot list for the S3 scenario shows 09:25 · out-of-area shows the request-a-quote chip.
**On camera:** "Why is 9:00 gone?" tooltip. Fixed-hour slots vs dynamic slots side by side.
**Clips:** "Fixed hourly slots waste his day. Dynamic slots fill it." · "Property tests find bugs I'd never write a case for."

---

## Phase 4 — Database + driver console (Ep 7–8)

> Phase 4. Load the `demo-mode-seams` skill. Add Drizzle + Neon schema per `docs/ARCHITECTURE.md` §3 with an in-memory fallback seeded from fixtures. Build the driver PWA at `/drive`: Auth.js v5 email allowlist (dev-only sign-in button in demo mode), day timeline with gap and deadhead minutes computed by the engine, availability (default 05:00–24:00), one-off blocks (S8), and installable manifest. Wire `POST /api/bookings` with the advisory lock and re-run of `canInsert` in the transaction, plus the 5-minute hold. Add an integration test for S9 (two concurrent requests, one wins).

**Acceptance:** S9 passes · timeline matches engine output · availability/blocks change slot results immediately.
**On camera:** the double-booking bug you'd have without the lock.
**Clip:** "Two riders tap the same slot at the same time. Here's what actually happens."

---

## Phase 5 — Tracking, ETA, T-30 gate, late risk (Ep 9–10)

> Phase 5. Load the `realtime-tracking` skill. Implement the driver location drop: foreground `watchPosition` (throttled) plus a ping on every status tap, stored in Redis with a 120 s TTL (in-memory fallback). Build `GET /api/rides/[token]`: countdown before T-30, live location + ETA inside the window, `403 not_yet_visible` otherwise, "last seen N s ago" for stale pings. Implement `projectDay` late-risk and cascade (L1, L2), surfacing alerts in the driver console and emitting an event for the async layer. Add a "simulate driver" dev tool that replays a route so this is demoable at a desk.

**Acceptance:** V1, L1, L2 pass · API test proves no location leaks before T-30 · stale ping flagged · simulator drives the rider screen.
**On camera:** iOS PWA limits, honestly. Show what happens when the driver locks the phone.
**Clips:** "Location is only visible 30 minutes out. Here's the API rule that enforces it." · "How the app knows he's going to be late before he does."

---

## Phase 6 — Payments and policy (Ep 11) — "Policy to Code"

> Phase 6. Load the `demo-mode-seams` skill. Implement `lib/payments` with Stripe **test mode** (PaymentIntent at hold, webhook `held → confirmed`, idempotent on event id) and a simulated fallback. Implement `lib/policy` from `policy.config.ts` (D-52, D-53) with tests C1–C5, and rider cancel via `POST /api/rides/[token]/cancel`. Do not enable live mode. If you are unsure about a Stripe API detail, check the current docs via MCP and cite it in a code comment.

**Acceptance:** C1–C5 pass · webhook idempotency test · a held booking that never pays expires and frees its slot.
**On camera:** the "Policy to Code" format: a written policy in a table becomes a config file and a test suite. Tie to `COMPLIANCE_CHECKLIST.md` (what's a business rule vs a legal rule).
**Clip:** "A cancellation policy is just a function. Here's the whole thing."

---

## Phase 7 — n8n async layer (Ep 12)

> Phase 7. Load the `n8n-async-layer` skill. Create workflows under `n8n/workflows/`: booking-confirmed, t-minus-30 reminder (schedule trigger → `GET /api/n8n/due` → send → `POST /api/n8n/ack`), late-risk notice, no-show follow-up, and a daily Slack digest. Use the `Authorization: Bearer` header exactly. Enforce idempotency with the `notifications.idempotency_key`. Demo mode writes to `/dev/outbox`. Export each workflow as JSON and document credential setup, including project-level sharing vs ownership.

**Acceptance:** duplicate delivery sends once · wrong Bearer returns 401 · outbox page shows every message · nothing in the live path awaits n8n.
**On camera:** the credential-sharing silent failure, on purpose.
**Clip:** "Why the ETA screen never touches n8n."

---

## Phase 8 — Harden, verify, ship, write it up (Ep 13)

> Phase 8. Run the `pre-deploy-auditor` agent and fix everything it finds. Add one Playwright happy path (book → pay (test) → driver marks complete → receipt). Add Sentry, PostHog, and Vercel Analytics behind env checks. Run an accessibility pass on `/book`, `/ride/[token]`, `/drive`. Load test the slot search against the provider-call budget. Write `docs/CASE_STUDY.md` (problem, architecture, honest limitations, what a second driver would change, "same engine, different business") using only facts that exist in the repo: no invented metrics. Update `projects.csv`-style metadata in `build.config.ts`. Deploy to the Vercel preview and production subdomain in **test mode**.

**Acceptance:** auditor clean · e2e green · a11y issues fixed or listed · case study contains no fabricated numbers.
**On camera:** the audit, including what it found that you missed.
**Clips:** "The reviewer agent found three things I'd have shipped." · "One build, three doors: the case study, the module, the episode."

---

## After the build

- **Employment door:** case study + repo + 2-minute walkthrough.
- **Consulting door:** extract `lib/scheduler` and `lib/routing` as a starter module; write a one-page offer for solo operators who travel between appointments.
- **Brand door:** assemble a "Build 032" playlist from the episode notes.
- **Before any real launch:** work through `docs/COMPLIANCE_CHECKLIST.md`.
