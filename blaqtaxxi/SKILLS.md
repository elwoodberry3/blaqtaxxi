# SKILLS.md — index

Claude Code loads skills from **`.claude/skills/<name>/SKILL.md`** (one folder per skill, YAML frontmatter). There is no special root file it reads by this name, so this page is a human-readable index. Claude finds the real skills from their `description` lines.

| Skill | Load when | Phases |
|---|---|---|
| [`layout-system`](.claude/skills/layout-system/SKILL.md) *(supplied; read `manifest.json` first)* | Building or reviewing any screen. Amendments: `docs/LAYOUT_AMENDMENTS.md` | 0.5, 3–9 |
| [`scheduling-engine`](.claude/skills/scheduling-engine/SKILL.md) | `lib/scheduler` (incl. car windows), `lib/routing`, `lib/pricing` (per car), slot search, late-risk | 1, 2, 3, 5 |
| [`admin-backend`](.claude/skills/admin-backend/SKILL.md) | `/admin`: availability and overrides, cars + assignments, price configs, media, policy, copy, config export/import, audit log | 4, 6, 7 |
| [`driver-shell`](.claude/skills/driver-shell/SKILL.md) | The thin native app for the driver (Android + iPhone): background location, device registration, spike protocol | 0.5, 5 |
| [`realtime-tracking`](.claude/skills/realtime-tracking/SKILL.md) | Driver navigation flow, pings, customer trip page, customer link scheme, T-30 gate, simulator | 0.5, 3, 5 |
| [`n8n-async-layer`](.claude/skills/n8n-async-layer/SKILL.md) | Workflows, `/api/n8n/*`, notifications, idempotency, pilot messaging rules, feedback workflow | 7 |
| [`demo-mode-seams`](.claude/skills/demo-mode-seams/SKILL.md) | External integrations, `APP_ENV` (demo/pilot/production), chips (`demo` only), pilot banner | 0, 2, 4, 6, 7, 9 |
| [`course-episode-capture`](.claude/skills/course-episode-capture/SKILL.md) | End of every phase (`demo` material only) | all |

## Agents (independent, read-only reviewers)

| Agent | Use |
|---|---|
| [`scheduler-verifier`](.claude/agents/scheduler-verifier.md) | Before calling any scheduler, pricing, or policy work done |
| [`pre-deploy-auditor`](.claude/agents/pre-deploy-auditor.md) | End of Phase 8, before opening the pilot to the public (Phase 9), and before the client's go-live (Phase 10) |

## Slash commands

| Command | Does |
|---|---|
| `/start-phase <n>` | Runs a phase from `docs/BUILD_PLAN.md` with gates (0, 0.5, 1–10) |
| `/wireframe <name>` | Matches a layout-system wireframe via the manifest and converts it to a component |
| `/brand-audit` | Raw hex, non-token colors, gray-as-text, red/amber misuse, contrast, a11y, chips |
| `/verify-scheduler` | Purity grep + tests + verifier agent |
| `/capture-episode <n>` | Writes the Skool episode notes |
| `/log-decision <text>` | Adds to `docs/DECISIONS.md` |

## Where things live

| Need | File |
|---|---|
| Rules and constraints (imports `CLAUDE.layout.md`) | `CLAUDE.md` |
| **What's still open, what it blocks, asset findings** | `docs/UNKNOWNS.md` |
| Answered questions and labeled assumptions | `docs/DECISIONS.md` |
| What each side sees, delivery model, scope, stories | `docs/PRD.md` |
| Every screen, by audience | `docs/SCREEN_MAP.md` |
| The hard part | `docs/SCHEDULER_SPEC.md` |
| System shape, data, API, environments | `docs/ARCHITECTURE.md` |
| Review of the supplied layout system | `docs/LAYOUT_AMENDMENTS.md` |
| Order of work and prompts | `docs/BUILD_PLAN.md` |
| Pilot process and approval | `docs/PILOT_PLAYBOOK.md` |
| Handoff skeleton | `docs/CLIENT_DEPLOY_GUIDE.md` |
| Tokens, contrast, typography, logo | `docs/BRAND.md` |
| Launch gate (unverified) | `docs/COMPLIANCE_CHECKLIST.md` |
| MCP servers | `.mcp.json`, `docs/MCP_SETUP.md` |
| Source notes, client brief, answers | `docs/DISCOVERY_TRANSCRIPT.md` |
| Test data | `docs/fixtures/travel-matrix.json`, `docs/fixtures/vehicles.json` |
| Client originals (gitignored) | `client-assets/` |
| Visual review of layouts | `docs/layout-preview/preview.html` |
