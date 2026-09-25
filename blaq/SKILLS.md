# SKILLS.md — index

Claude Code loads skills from **`.claude/skills/<name>/SKILL.md`** (one folder per skill, with YAML frontmatter). There is no special root file that it reads by that name, so this page is a human-readable index for the course. Claude discovers the real skills automatically from their `description` lines.

| Skill | Load when | Phases |
|---|---|---|
| [`scheduling-engine`](.claude/skills/scheduling-engine/SKILL.md) | Anything in `lib/scheduler`, `lib/routing`, `lib/pricing`, slot search, traffic, late-risk | 1, 2, 3, 5 |
| [`realtime-tracking`](.claude/skills/realtime-tracking/SKILL.md) | Driver pings, rider ride page, ETA, T-30 gate, simulator | 5 |
| [`n8n-async-layer`](.claude/skills/n8n-async-layer/SKILL.md) | Workflows, `/api/n8n/*`, notifications, idempotency | 7 |
| [`demo-mode-seams`](.claude/skills/demo-mode-seams/SKILL.md) | Any external integration; TodoChip/StatusChip | 0, 2, 4, 6, 7 |
| [`course-episode-capture`](.claude/skills/course-episode-capture/SKILL.md) | End of every phase | all |

## Agents (independent reviewers, read-only)

| Agent | Use |
|---|---|
| [`scheduler-verifier`](.claude/agents/scheduler-verifier.md) | Before calling any scheduler work done |
| [`pre-deploy-auditor`](.claude/agents/pre-deploy-auditor.md) | End of Phase 8 and before any production deploy |

## Slash commands

| Command | Does |
|---|---|
| `/start-phase <n>` | Runs a phase from `docs/BUILD_PLAN.md` with gates |
| `/verify-scheduler` | Purity grep + tests + verifier agent |
| `/capture-episode <n>` | Writes the Skool episode notes |
| `/log-decision <text>` | Adds to `docs/DECISIONS.md` |

## Where things live

| Need | File |
|---|---|
| Rules and constraints | `CLAUDE.md` |
| Answers to questions | `docs/DECISIONS.md` |
| What and why | `docs/PRD.md` |
| The hard part | `docs/SCHEDULER_SPEC.md` |
| System shape | `docs/ARCHITECTURE.md` |
| Order of work and prompts | `docs/BUILD_PLAN.md` |
| Tokens and voice | `docs/BRAND.md` |
| Launch risks (unverified) | `docs/COMPLIANCE_CHECKLIST.md` |
| MCP servers | `.mcp.json`, `docs/MCP_SETUP.md` |
| Source notes | `docs/DISCOVERY_TRANSCRIPT.md` |
| Test numbers | `docs/fixtures/travel-matrix.json` |
