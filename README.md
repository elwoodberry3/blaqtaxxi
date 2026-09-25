![BLAQ](./assets/imgs/hero.jpg)
# BLAQ  
A one-driver, one-fare ride-booking app, specified so Claude Code can build it from files instead of chat. This is the **scaffold**: docs, rules, skills, reviewer agents, slash commands, MCP config. No app code yet; Phase 0 creates it.

**The hard problem:** not dispatch, but *travel-time-aware scheduling* for a single car. A ride is only bookable if the driver can reach it from where his previous ride ends, in the traffic at that hour, and still make his next pickup.

## Use it

1. Create an empty repo (`elwoodberry3/blaqtaxxi`) and copy this folder's contents into the root, including the hidden `.claude/` directory and `.mcp.json`.
2. Export MCP secrets in your shell (see `docs/MCP_SETUP.md`), then launch Claude Code from the repo root.
3. Paste the **Episode 0 prompt** from `docs/BUILD_PLAN.md`. Claude reads the docs and reports its riskiest assumptions. It should not write code.
4. Say "go", then run `/start-phase 0`. Repeat per phase. Each phase stops at a gate for your review.
5. After each phase, `/capture-episode <n>` produces the Skool notes and clip candidates.

## What's inside

```
CLAUDE.md                 governance: rules, gotchas, working agreement
SKILLS.md                 index of skills, agents, commands
.mcp.json                 GitHub, Neon, Vercel, Stripe(test), n8n, Playwright, docs
.env.example              every variable optional; no keys = demo mode
.claude/
  settings.json           permissions (deny .env reads, force-push, rm -rf)
  skills/                 scheduling-engine, realtime-tracking, n8n-async-layer,
                          demo-mode-seams, course-episode-capture
  agents/                 scheduler-verifier, pre-deploy-auditor (read-only)
  commands/               start-phase, verify-scheduler, capture-episode, log-decision
docs/
  DISCOVERY_TRANSCRIPT.md verbatim notes + annotations
  DECISIONS.md            answered questions, tagged stated / inferred / assumed
  PRD.md                  scope, stories, risks
  SCHEDULER_SPEC.md       algorithm + test cases with exact numbers
  ARCHITECTURE.md         real-time vs async, data model, states, API, env matrix
  BUILD_PLAN.md           9 phases, 13 episodes, paste-ready prompts, gates
  BRAND.md                IAS tokens, swappable
  COMPLIANCE_CHECKLIST.md unverified launch questions
  MCP_SETUP.md            server notes and cautions
  fixtures/travel-matrix.json  illustrative DFW travel times for tests/demo
```

## Read these caveats first

- **Assumptions are labeled.** `docs/DECISIONS.md` tags each answer `stated`, `inferred` or `assumed`. The "Open" list at the bottom is the only set of things that should need your input.
- **Build 032, repo name and subdomain** are assumed from `projects.csv` patterns.
- **"Louisville, Lil' M … $25"** is read as **Lewisville** → Dallas. Confirm.
- **Travel times in the fixtures are illustrative**, not measured traffic.
- **Legal, insurance, permit and privacy status is unverified.** See the compliance checklist before any real launch. This is not legal advice.
- **MCP servers were not launched** while building this scaffold; verify with `/mcp`.
- The IAS palette has **no warning/error color** (D-73). Late-risk UI uses icon + text until you approve one.
- Claude Code reads skills from `.claude/skills/*/SKILL.md`; the root `SKILLS.md` is only an index.
