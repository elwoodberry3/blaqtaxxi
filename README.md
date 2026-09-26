![](./assets/imgs/hero.jpg)
# BLAQTAXXI — Claude Code project scaffold

A production-grade build for a client (the owner-driver of BLAQTAXXI), taught episodically in an advanced Skool course. One driver, one active fare at a time. He owns several cars (a black Nissan Sentra today, a luxury SUV as an example), **customers choose the car at booking**, and each car has its own price configuration. Specified so Claude Code can build it from files instead of chat.

This is the **scaffold**: docs, rules, skills, reviewer agents, slash commands, the client's layout system and brand assets, and MCP config. No app code yet; Phase 0 creates it.

**The hard problem:** not dispatch, but *travel-time-aware scheduling* for a single driver. A ride is only bookable if the driver can reach it from where his previous ride ends, in the traffic at that hour, in a car that is assigned then, and still make his next pickup.

**Three surfaces:** an **admin backend** (`/admin`) that configures what customers see; a **driver app** (`/drive` in a thin native shell for Android and iPhone) that opens Google Maps to the pickup and then to the drop-off; and a **link-based customer page** (`/pickup/johnson-4821-k9Xp2mQz`) with before / during / after states. No customer accounts.

**Delivery:** a **pilot** on IAS's accounts (`blaqtaxxi.iasbootcamp.com`), where the client kicks the tires for 1–2 feedback rounds and the public can go right up to the payment step; then, once he approves, he deploys it on **his own** domain and accounts from IAS's instructions. No real payment is taken in the pilot.

## Use it

1. **Unzip into the root of the repo.** The zip has **no wrapper folder**, so the files land at the root, including the hidden `.claude/`, `.mcp.json`, `.gitignore`, and `.githooks/`. If you ever see a nested `blaqtaxxi/` folder, flatten it **including dotfiles** (`.claude`, `.mcp.json`, `.gitignore`), because `.claude/settings.json` and the skills only apply from the folder you launch Claude Code in.
2. **Before your first commit:** put the client's images in `client-assets/` on your machine only (it is gitignored, and `.githooks/pre-commit` blocks it; run `git config core.hooksPath .githooks`). **The repo should be private.** Check `git ls-files` shows no image before you push.
3. Export MCP secrets in your terminal (see `docs/MCP_SETUP.md`), then, from the repo root, start Claude Code in **plan mode**: `claude --permission-mode plan`.
4. Paste the **Episode 0 prompt** from `docs/BUILD_PLAN.md`. Claude reads the docs, reports its riskiest assumptions and which BLOCKING unknowns apply, and presents a Phase 0 plan. No files change in plan mode.
5. Approve the plan to start Phase 0. Then `/start-phase <n>` for each later phase (0, 0.5, 1–10). Each phase stops at a gate for your review.
6. After each phase, `/capture-episode <n>` produces the Skool notes and clip candidates (from `demo` material only).

**About `/mcp`:** a command you type *inside* a running Claude Code session, at its prompt box, not in your terminal shell. It lists the MCP servers from `.mcp.json`, shows which are connected, and lets you sign in to the ones that use a browser login (Neon, Vercel). Outside a session, `claude mcp list` shows the same servers.

## Before you fire the first prompt

Read `docs/UNKNOWNS.md`, Section 1. Nothing stops the plan-mode prompt. Two things are still open: whether IAS has a **physical iPhone and a Mac** for the iOS drive test and TestFlight builds (U-D7), and a one-word confirmation of the placeholder SUV prices (U-V5).

## What's inside

```
CLAUDE.md                 governance (imports CLAUDE.layout.md): rules, gotchas, working agreement
CLAUDE.layout.md          the supplied BLAQ token and layout rules (unmodified)
SKILLS.md                 index of skills, agents, commands
.mcp.json                 GitHub, Neon, Vercel, Stripe(test), n8n, Playwright, docs
.env.example              APP_ENV-aware (demo | pilot | production)
client-assets/            README only; the client's originals go here LOCALLY (gitignored, guarded)
.githooks/pre-commit      blocks client images from being committed
.claude/
  settings.json           permissions (deny .env reads, force-push, rm -rf)
  skills/                 layout-system (supplied, unmodified), scheduling-engine, admin-backend,
                          realtime-tracking, driver-shell, n8n-async-layer, demo-mode-seams,
                          course-episode-capture
  agents/                 scheduler-verifier, pre-deploy-auditor (read-only)
  commands/               start-phase, wireframe, brand-audit, verify-scheduler,
                          capture-episode, log-decision
docs/
  UNKNOWNS.md             what is open, what it blocks, defaults, spikes, asset findings
  DECISIONS.md            answered questions: stated / inferred / assumed / approved
  PRD.md                  what each side sees, delivery model, scope, stories, risks
  SCREEN_MAP.md           every customer, driver, and admin screen and its layout source
  SCHEDULER_SPEC.md       algorithm + test cases with exact numbers (S, L, V, P, C)
  ARCHITECTURE.md         real-time vs async, data model, states, API, environments
  LAYOUT_AMENDMENTS.md    review of the supplied layout system and our deviations
  BUILD_PLAN.md           phases 0, 0.5, 1–10, paste-ready prompts, gates
  PILOT_PLAYBOOK.md       kick-the-tires process, feedback rounds, approval
  CLIENT_DEPLOY_GUIDE.md  SKELETON of the handoff guide (unverified until Phase 10)
  BRAND.md                BLAQ tokens, measured contrast, typography, logo rules
  COMPLIANCE_CHECKLIST.md unverified launch gate
  MCP_SETUP.md            server notes and cautions
  DISCOVERY_TRANSCRIPT.md source notes, client brief, and Steve's answers
  fixtures/               travel-matrix.json, vehicles.json (fake / illustrative)
  layout-preview/         preview.html + the original layout-system README
```

## Read these caveats first

- **Production wins over teaching.** No client or customer data on camera; the course is recorded on `demo` with fake data; nothing is published without the client's consent (U-C1).
- **The driver app needs a native shell.** Verified: Google's Navigation SDK is native-only and Maps URLs start navigation without an API key. Approved by Steve: a thin native shell; **iPhone is the default** (TestFlight internal testing; the iPhone is not real yet, so we plan as if it is) with Android supported. IAS never publishes the app. **Not verified:** how a background-location plugin behaves on his real phones, and Apple/Google distribution rules and costs. Phase 0.5 runs the drive tests. iOS builds need a Mac, Xcode, and an Apple developer account, which Claude cannot do from a cloud session.
- **Brand assets have limits.** The wordmark PNG is 195×75 and opaque (it cannot go on navy); the favicon is a soft 512 px JPEG; the driver photo is a tilted, distorted selfie; the car photos look like manufacturer stock images. Details and fixes in `docs/UNKNOWNS.md` §3.
- **Momo Trust Display** is confirmed as a Google Font; its weights and license could not be verified from here.
- **Approved:** the customer link with a random suffix, per trip; canvas colors; `blaq-gray-text #686D71`; a warning color (the hex `#A15C00` is my proposal); the native shell; customer car choice.
- **Still my reading or my picks:** the cutoff rule (D-09); cancellation dollar amounts (D-52); the Suburban's seats and prices (it is an example car; placeholder $65 / $85 / $110, anchored to unverified third-party Uber Black estimates).
- **IAS never processes the client's real fares** (confirmed): its Stripe is always test mode. Stripe is test mode in the pilot; his own live account is used in his own deployment.
- **Unverified:** Google Maps Platform pricing, SMS registration timing, Stripe fees, storage limits, and every legal, insurance, permit, and privacy requirement (a launch gate).
- Travel times in fixtures are illustrative, not measured traffic. MCP servers were not launched while building this scaffold; verify with `/mcp`.
- Claude Code reads skills from `.claude/skills/*/SKILL.md`; `SKILLS.md` is only an index. Confirm `@CLAUDE.layout.md` import support on your Claude Code version; if it does not expand, paste that file into `CLAUDE.md`.
