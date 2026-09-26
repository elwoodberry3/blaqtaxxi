![](./assets/imgs/hero.jpg)
# BLAQTAXXI — Claude Code project scaffold

A production build for a paying client, taught episodically in an advanced Skool course. One driver, one active fare at a time, one car today and possibly many later, each with its own price configuration. Specified so Claude Code can build it from files instead of chat.

This is the **scaffold**: docs, rules, skills, reviewer agents, slash commands, the client's layout system, and MCP config. No app code yet; Phase 0 creates it.

**The hard problem:** not dispatch, but *travel-time-aware scheduling* for a single driver. A ride is only bookable if the driver can reach it from where his previous ride ends, in the traffic at that hour, and still make his next pickup.

**Three surfaces:** an **admin backend** (`/admin`) that configures what customers see, a **driver console** (`/drive`) with Google Maps navigation to the pickup and then to the drop-off, and a **link-based customer page** (`/pickup/…`) with before / during / after states. No customer accounts.

## Use it

1. Create an empty repo (`elwoodberry3/blaqtaxxi`) and copy this folder's contents into the root, including the hidden `.claude/` directory, `.mcp.json`, and `CLAUDE.layout.md`.
2. Export MCP secrets in your terminal (see `docs/MCP_SETUP.md`), then, from the repo root, start Claude Code in **plan mode**: `claude --permission-mode plan`.
3. Paste the **Episode 0 prompt** from `docs/BUILD_PLAN.md`. Claude reads the docs, reports its riskiest assumptions and which BLOCKING unknowns apply, and presents a Phase 0 plan. No files change in plan mode.
4. Approve the plan to start Phase 0. Then `/start-phase <n>` for each later phase. Each phase stops at a gate for your review.
5. After each phase, `/capture-episode <n>` produces the Skool notes and clip candidates.

**About `/mcp`:** a command you type *inside* a running Claude Code session, at its prompt box, not in your terminal shell. It lists the MCP servers from `.mcp.json`, shows which are connected, and lets you sign in to the ones that use a browser login (Neon, Vercel). Outside a session, `claude mcp list` shows the same servers.

## Before you fire the first prompt

Read `docs/UNKNOWNS.md`, Section 1. Nothing stops the plan-mode prompt, but six answers change what Claude plans: the driver's phone and app form factor (U-D1), what "many cars" means (U-V1) and whether customers pick the car (U-V2), approval of the safer customer-link scheme (U-L1), the client's consent and account ownership (U-C1), and the brand assets and approvals (U-B1).

## What's inside

```
CLAUDE.md                 governance (imports CLAUDE.layout.md): rules, gotchas, working agreement
CLAUDE.layout.md          the supplied BLAQ token and layout rules (unmodified)
SKILLS.md                 index of skills, agents, commands
.mcp.json                 GitHub, Neon, Vercel, Stripe(test), n8n, Playwright, docs
.env.example              APP_ENV-aware; demo needs no keys, production requires them
.claude/
  settings.json           permissions (deny .env reads, force-push, rm -rf)
  skills/                 layout-system (supplied, unmodified), scheduling-engine, admin-backend,
                          realtime-tracking, n8n-async-layer, demo-mode-seams, course-episode-capture
  agents/                 scheduler-verifier, pre-deploy-auditor (read-only)
  commands/               start-phase, wireframe, brand-audit, verify-scheduler,
                          capture-episode, log-decision
docs/
  UNKNOWNS.md             what is open, what it blocks, defaults, spikes
  DECISIONS.md            answered questions: stated / inferred / assumed
  PRD.md                  what each side sees, scope, stories, risks
  SCREEN_MAP.md           every customer, driver, and admin screen and its layout source
  SCHEDULER_SPEC.md       algorithm + test cases with exact numbers (S, L, V, P, C)
  ARCHITECTURE.md         real-time vs async, data model, states, API, environments
  LAYOUT_AMENDMENTS.md    review of the supplied layout system and our deviations
  BUILD_PLAN.md           phases 0, 0.5, 1–9, paste-ready prompts, gates
  BRAND.md                BLAQ tokens, measured contrast, proposed fixes
  COMPLIANCE_CHECKLIST.md unverified launch gate
  MCP_SETUP.md            server notes and cautions
  DISCOVERY_TRANSCRIPT.md source notes + the client brief update
  fixtures/               travel-matrix.json, vehicles.json (fake, illustrative)
  layout-preview/         preview.html + the original layout-system README
```

## Read these caveats first

- **Production wins over teaching.** No client data on camera; the course is recorded on `demo`/`staging`; nothing is published without the client's consent (U-C1).
- **The layout system has real problems** (contrast failures, rider-account screens that do not fit a link-based customer, no driver or admin screens). See `docs/LAYOUT_AMENDMENTS.md`. The original files are copied unmodified.
- **The biggest technical risk is live location while the driver uses Google Maps.** Verified: Google's Navigation SDK is native-only and Maps URLs start navigation without an API key. Not verified: whether a web app keeps reporting location once the driver is in the Maps app. Spike SP-1 on the client's phone decides (U-D1).
- **The customer link as specified is guessable.** A safer scheme is proposed and needs approval (U-L1).
- **Confirmed:** Build 032, Lewisville, TX, price tiers. **Still my reading:** the cutoff rule (D-09). **My picks, not published by Uber or Lyft:** cancellation dollar amounts (D-52).
- **Not received:** `nissan-sentra.jpg`.
- **Unverified:** Google Maps Platform pricing, SMS carrier-registration timing, Stripe's current fees, and every legal, insurance, permit, and privacy requirement (a launch gate).
- **Travel times in fixtures are illustrative**, not measured traffic. MCP servers were not launched while building this scaffold; verify with `/mcp`.
- Claude Code reads skills from `.claude/skills/*/SKILL.md`; the root `SKILLS.md` is only an index. `@CLAUDE.layout.md` import support should be confirmed on your Claude Code version; if it does not expand, paste that file into `CLAUDE.md`.
