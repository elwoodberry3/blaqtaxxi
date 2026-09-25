---
description: Start a numbered build phase from docs/BUILD_PLAN.md (usage /start-phase 1)
argument-hint: <phase number 0-8>
---

Start **Phase $ARGUMENTS**.

1. Re-read `CLAUDE.md` and the Phase $ARGUMENTS section of `docs/BUILD_PLAN.md`. Read the current repo state first (repo-first; never approximate from memory).
2. Load every skill the phase names (`.claude/skills/<name>/SKILL.md`).
3. In two or three lines, tell me what you're about to do and which spec ids / decision ids apply. Do not ask questions already answered in `docs/DECISIONS.md`. Ask (with AskUserQuestion) only if something is architectural or irreversible and not covered.
4. Create a task list for the phase, ending with a verification task.
5. Build. Tests first for anything in `lib/scheduler`, `lib/pricing`, `lib/policy`.
6. Prove it: run `npm run typecheck && npm run lint && npm test` and the phase's acceptance checks; show real output.
7. Run `/capture-episode $ARGUMENTS`.
8. Stop at the gate and tell me what to review. Do not start the next phase until I say go.
