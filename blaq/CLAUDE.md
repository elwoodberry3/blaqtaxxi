# CLAUDE.md — BLAQTAXXI ("One-to-One Uber")

> Governance file for Claude Code. Read this first, every session. If this file and a chat message disagree, follow the chat message, then tell me so this file can be updated.

**Build:** IAS Build 032 (assumed; `projects.csv` ends at 031, confirm in `docs/DECISIONS.md` D-01)
**Repo:** `elwoodberry3/blaqtaxxi` · **Subdomain:** `blaqtaxxi.elwoodberry.com` · **Category:** Agentic Service Operations
**Purpose:** Advanced Skool course build (Claude Code), directed from the discovery notes in `docs/DISCOVERY_TRANSCRIPT.md`.
**Governing principle:** One Build. Three Doors. Every phase should yield (1) a portfolio/case-study asset, (2) a reusable consulting module, (3) a recordable episode.

---

## 1. What this product is (one paragraph)

One car. One driver. One fare at a time. Riders **reserve a time**, pay a **flat fee** up front, and can see when the driver is coming. The driver sees a day schedule and fills it with as many rides as physically possible. The core problem is **not dispatch, it is travel-time-aware scheduling**: a new booking is only offered if the driver can get there on time from wherever his *previous ride ends*, in *the traffic that will exist at that hour*, and still make his *next* pickup after it.

## 2. Read order (before writing any code)

1. `docs/DECISIONS.md` — answered questions, assumptions, and the short list still open. **Do not re-ask anything answered here.**
2. `docs/PRD.md` — scope, user stories, MVP vs later.
3. `docs/SCHEDULER_SPEC.md` — the heart of the build. Rules, algorithm, test cases.
4. `docs/ARCHITECTURE.md` — real-time vs async split, data model, state machine, API surface.
5. `docs/BUILD_PLAN.md` — phases, prompts, acceptance gates.
6. `docs/BRAND.md` — tokens (IAS system, swappable).
7. `SKILLS.md` — index of `.claude/skills/*`. Load the relevant skill before working in its area.

## 3. Non-negotiables (hard constraints)

1. **Baseline Stack + DXP.** Next.js 14 (App Router) · TypeScript (strict) · Tailwind · Vercel · cloud-hosted n8n (`https://iautomateshit.app.n8n.cloud`). Composable, decoupled frontend/backend, modular payment/auth/integration subsystems, shared design tokens.
2. **Real-time vs async split.** The live path (location pings → ETA → rider screen) **never touches n8n**. n8n owns only the async layer: confirmations, T-30 reminders, late-ride notices, CRM/Slack sync.
3. **The scheduler is a pure module.** `lib/scheduler/**` has **zero** imports from Next, the DB, Stripe, or any network client. Inputs in, decision out. Travel time is injected via a `TravelTimeProvider` interface. This is what makes it testable and reusable for consulting clients.
4. **Demo mode first.** Every external integration (routing, Stripe, Resend, SMS, n8n, Neon, Redis) sits behind an env-var check with a graceful demo fallback. `npm run dev` with **zero credentials** must give a fully clickable app.
5. **Visible gaps.** Incomplete items render `<TodoChip>` / `<StatusChip>` on the page. Never hide unfinished work. The gaps are part of the demonstration.
6. **No fabricated numbers.** Travel times in demo mode are an illustrative model and are labeled as such in the UI and code. No invented revenue, ride counts, ratings, or testimonials anywhere (see Brand/AI governance).
7. **No real personal data in the repo.** Fixtures use fake riders and fake addresses. Do not commit the real driver's phone, handle, or plate. Secrets live in `.env.local` only.
8. **Stripe stays in test mode** until Steve explicitly says otherwise in writing.
9. **Ask before irreversible moves:** live-mode payments, sending real SMS/email to real people, deleting data, force-pushing, anything that spends money via a paid API beyond the free/dev tier.

## 4. Domain rules you must not get wrong

| Rule | Value | Source |
|---|---|---|
| Fleet | 1 driver, 1 vehicle, 1 active ride at a time | stated |
| Service area | Dallas–Fort Worth (DFW) | stated |
| Base fare | **$20 flat** anywhere in DFW | stated |
| Long-trip fare | trips over ~30 min: **$25–$30** | stated (tier split assumed, D-07) |
| Availability | every day **05:00–24:00** America/Chicago | stated |
| Booking lead time | prefer 1–2 h ahead; "ride now" allowed if feasible | stated/inferred |
| Rider visibility | countdown always; **live location + ETA only from T-30 min** | stated |
| Payment | book → pay → flat fee | stated |
| Pricing vs traffic | rush hour affects **scheduling**, not **price** | assumed (D-07) |

**The one sentence that defines correctness:** *A booking is valid only if `prevRide.end + travel(prev.dropoff → new.pickup, at that time) + buffer ≤ new.pickup` AND `new.end + travel(new.dropoff → next.pickup, at that time) + buffer ≤ next.pickup`, all inside availability.* Rush hour makes `travel(...)` bigger; a drop-off "around the corner" makes it tiny, so the next pickup can be minutes later, not an hour later.

## 5. Repo layout (target)

```
app/
  (rider)/book/            slot picker, quote, checkout
  (rider)/ride/[token]/    countdown → live ETA → receipt (signed-link, no account)
  (driver)/drive/          day timeline, status buttons, location drop
  api/                     route handlers (see ARCHITECTURE.md)
lib/
  scheduler/               PURE. feasibility, slot search, ETA/late-risk. No I/O.
  routing/                 TravelTimeProvider: google.ts | demo.ts (traffic model)
  pricing/                 tier rules → quote
  policy/                  cancellation/no-show rules (policy.config.ts)
  tracking/                ping ingest, T-30 visibility gate
  payments/                Stripe seam + demo fallback
  notify/                  email/SMS seam → n8n webhooks
  db/                      Drizzle schema, queries, seed
components/                ui/, TodoChip, StatusChip, Timeline, EtaCard
n8n/workflows/             exported JSON, one file per workflow
tests/                     scheduler/ (the big one), pricing/, policy/, e2e/
docs/                      spec + course material
.claude/                   skills, agents, commands, settings
```

## 6. Coding conventions

- TypeScript `strict`. No `any` in `lib/scheduler`. Prefer discriminated unions for states.
- All times stored **UTC**, displayed **America/Chicago**. Never compute "5am–midnight" in UTC. Use a single `lib/time.ts`; write DST-day tests (Nov 1 and Mar 8 style days).
- Money is **integer cents**. Never floats.
- Tailwind uses tokens from `docs/BRAND.md` via CSS variables (`--ias-*` aliases). Kinetic Emerald is one use per view, dark text on it only. 60/30/10 ratio.
- Fonts: Space Grotesk / Space Mono via a runtime `<link>`, **not** `next/font/google` (see gotchas).
- Accessibility: WCAG AA contrast, real `<button>`s, focus rings, live regions for ETA changes.
- Every route handler validates input with Zod and returns typed errors.
- Small commits, one concern each. Conventional commit messages.

## 7. Known gotchas (learned the hard way on prior builds — do not relearn)

1. **`output: 'export'` silently kills API routes.** Never set it. This build has live server functions.
2. **`NEXT_PUBLIC_*` vars are inlined at build time.** Read them directly inside the component/function body, never through a module-level `const` chain, or production bundles can get `undefined`.
3. **`next/font/google` fails at build** when `fonts.googleapis.com` is blocked (sandboxes). Use a runtime `<link>` and note it in the README.
4. **n8n auth:** header key must be exactly `Authorization`, value must include the `Bearer ` prefix. Credential *ownership* vs project-level *sharing* causes silent auth failures — check both.
5. **Version drift:** never cherry-pick files from different patch drops. Apply full, consolidated patches.
6. **HubSpot Free ceiling:** 10 custom contact properties. If CRM sync is added, triage properties first.
7. **Vercel functions are not long-lived sockets.** Use short polling for rider ETA (10 s) and short POSTs for driver pings. Do not build WebSocket/SSE infrastructure for the MVP.
8. **iOS PWAs cannot track location in the background.** Driver location drops happen while the console is open, plus on every status tap. Design around this; do not promise continuous tracking.
9. **Routing APIs cost money per element.** Cache travel times by (origin grid cell, destination grid cell, hour bucket). Cap candidate slots per search.

## 8. Working agreement (how to work with me)

1. **Repo-first.** Before changing anything, read the actual current files. Never approximate from memory.
2. **Structured decisions.** If a decision is not in `docs/DECISIONS.md` and it is *architectural or irreversible*, stop and ask me with `AskUserQuestion` (2–4 options, recommended first). If it is cheap to reverse, decide, log it in `DECISIONS.md` as `assumed`, and keep moving.
3. **Phase gates.** Follow `docs/BUILD_PLAN.md`. Do not start Phase N+1 until Phase N's acceptance checks pass and I say go.
4. **Tests first for the scheduler.** Write the failing test from `SCHEDULER_SPEC.md`, then the code. No scheduler change merges without its test.
5. **Show, don't claim.** When you finish something, run it and show output (test run, screenshot, curl). "Should work" is not done.
6. **Narrate for the camera.** Keep explanations short and direct. If something fails, say so plainly and debug in the open; failures are teaching content, do not polish them out.
7. **After each phase** run `/capture-episode` so the lesson material is produced while it is fresh.

## 9. Definition of done (per phase)

- [ ] Acceptance checks in `BUILD_PLAN.md` pass, evidence shown
- [ ] `npm run typecheck && npm run lint && npm test` green
- [ ] Runs in demo mode with zero credentials
- [ ] New gaps show as `TodoChip`s; nothing silently stubbed
- [ ] `DECISIONS.md` updated for any new assumption
- [ ] Episode notes captured (`/capture-episode`)

## 10. Things not to do

- Do not add a second driver, dispatch matching, surge pricing, ratings, or chat. Out of scope (see PRD).
- Do not put the scheduler behind an API call to itself, or into n8n.
- Do not add libraries when 20 lines will do. Prefer simple.
- Do not describe legal/insurance/permit status as settled. It is **unverified** (see `docs/COMPLIANCE_CHECKLIST.md`).
- Do not sound corporate. Direct, clear, technical, honest, demonstrable.
