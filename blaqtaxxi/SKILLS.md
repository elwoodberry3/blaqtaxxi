# SKILLS.md — index

Claude Code loads skills from **`.claude/skills/<name>/SKILL.md`** (one folder per skill, YAML frontmatter). There is no special root file it reads by this name, so this page is a human-readable index. Claude finds the real skills from their `description` lines.

| Skill | Load when | Phases |
|---|---|---|
| [`layout-system`](.claude/skills/layout-system/SKILL.md) *(supplied; read `manifest.json` first)* | Building or reviewing any screen. Amendments: `docs/LAYOUT_AMENDMENTS.md` | 0.5, 3–8 |
| [`scheduling-engine`](.claude/skills/scheduling-engine/SKILL.md) | `lib/scheduler`, `lib/routing`, `lib/pricing` (per-car), slot search, late-risk | 1, 2, 3, 5 |
| [`admin-backend`](.claude/skills/admin-backend/SKILL.md) | `/admin`: availability and overrides, cars, price configs, policy, copy, audit log, roles | 4, 6, 7 |
| [`realtime-tracking`](.claude/skills/realtime-tracking/SKILL.md) | Driver navigation flow, location drop, customer trip page, customer link scheme, T-30 gate, simulator | 0.5, 3, 5 |
| [`n8n-async-layer`](.claude/skills/n8n-async-layer/SKILL.md) | Workflows, `/api/n8n/*`, notifications, idempotency | 7 |
| [`demo-mode-seams`](.claude/skills/demo-mode-seams/SKILL.md) | External integrations, environments, chips (demo/staging only) | 0, 2, 4, 6, 7 |
| [`course-episode-capture`](.claude/skills/course-episode-capture/SKILL.md) | End of every phase (demo/staging material only) | all |

## Agents (independent, read-only reviewers)

| Agent | Use |
|---|---|
| [`scheduler-verifier`](.claude/agents/scheduler-verifier.md) | Before calling any scheduler, pricing, or policy work done |
| [`pre-deploy-auditor`](.claude/agents/pre-deploy-auditor.md) | End of Phase 8, and before any production deploy (Phase 9) |

## Slash commands

| Command | Does |
|---|---|
| `/start-phase <n>` | Runs a phase from `docs/BUILD_PLAN.md` with gates |
| `/wireframe <name>` | Matches a layout-system wireframe via the manifest and converts it to a component |
| `/brand-audit` | Raw hex, non-token colors, gray-as-text, red misuse, contrast, a11y, chips |
| `/verify-scheduler` | Purity grep + tests + verifier agent |
| `/capture-episode <n>` | Writes the Skool episode notes |
| `/log-decision <text>` | Adds to `docs/DECISIONS.md` |

## Where things live

| Need | File |
|---|---|
| Rules and constraints (imports `CLAUDE.layout.md`) | `CLAUDE.md` |
| **What's still open, and what it blocks** | `docs/UNKNOWNS.md` |
| Answered questions and labeled assumptions | `docs/DECISIONS.md` |
| What each side sees, scope, stories | `docs/PRD.md` |
| Every screen, by audience | `docs/SCREEN_MAP.md` |
| The hard part | `docs/SCHEDULER_SPEC.md` |
| System shape, data, API, environments | `docs/ARCHITECTURE.md` |
| Review of the supplied layout system | `docs/LAYOUT_AMENDMENTS.md` |
| Order of work and prompts | `docs/BUILD_PLAN.md` |
| Tokens, contrast, voice | `docs/BRAND.md` |
| Launch gate (unverified) | `docs/COMPLIANCE_CHECKLIST.md` |
| MCP servers | `.mcp.json`, `docs/MCP_SETUP.md` |
| Source notes and client brief | `docs/DISCOVERY_TRANSCRIPT.md` |
| Test data | `docs/fixtures/travel-matrix.json`, `docs/fixtures/vehicles.json` |
| Visual review of layouts | `docs/layout-preview/preview.html` |
