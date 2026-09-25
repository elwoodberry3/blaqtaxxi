---
name: course-episode-capture
description: Use at the end of every build phase, or when asked to capture an episode, to turn the work just done into Skool course material: episode notes, clip candidates, failures worth showing, and reusable takeaways. Build first, publish second.
---

# course-episode-capture

Principle: **Build First. Publish Second.** Content only comes from work that actually happened in this repo. Never invent results, metrics, or things that didn't run.

## Output

Write `docs/episodes/NN-<slug>.md` (NN = phase/episode number). Keep it tight; this is a shooting script, not an essay.

```md
# Episode NN — <plain title>

**Phase:** <n> · **Repo state:** <commit sha or "uncommitted"> · **Date:** <today>

## What we built (3 bullets, concrete)
## The one idea to teach (1 sentence)
## Demo path (numbered steps that work right now; include the exact commands)
## Failures and fixes (real ones only)
- What broke → why → fix. Keep unpolished. Failures are teaching content.
## Clip candidates (30–50 s, standalone, each with a hook line and the exact screen/command to show)
1. Hook: "…"  ·  Show: …  ·  Payoff: …
2. …
## Gotchas learned (add to CLAUDE.md §7 if new)
## Honest limitations (what this doesn't do yet; TodoChips visible in the demo)
## Reusable module (consulting door): <what could be lifted out and where it applies>
## Career angle (employment door): <the sentence a hiring manager would care about, supported by what exists>
## Links: spec ids covered (S/L/V/P/C), decisions touched (D-xx)
```

## Rules

1. Only cite tests that ran and commands that worked; paste actual output snippets, not paraphrases.
2. Prefer short, direct language. No hype, no "game-changer," no guru voice.
3. Each clip candidate must stand alone with no prior context and end on a concrete payoff. 30–50 seconds when spoken.
4. If a phase surfaced a new gotcha or assumption, append it to `CLAUDE.md` §7 or `docs/DECISIONS.md` in the same turn.
5. If a phase has a "Policy to Code" angle (a written rule turned into config plus tests), say so in the notes; that is a recurring series format.
6. No real rider or driver data in notes or screenshots. Use fixtures.
7. Never fabricate metrics (rides, revenue, ratings, time saved). If there's no measurement, say "not measured."

## Also produce

A three-line **LinkedIn-style summary** at the bottom of the episode file: problem, what was built, one honest limitation. No emojis, no buzzwords.
