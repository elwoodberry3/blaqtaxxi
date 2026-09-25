---
description: Purity grep, full test run, then independent scheduler-verifier review
---

Verify the scheduling engine.

1. Run the purity check and show the result (it must be empty):
   `grep -rnE "from '(next|drizzle-orm|stripe|@upstash)|process\.env|fetch\(|Date\.now\(|new Date\(\)|Math\.random" lib/scheduler --include=*.ts --exclude=*.test.ts`
2. Run `npm run typecheck && npm test` and show trimmed output.
3. Print a table of spec cases S1–S10, L1–L2, V1, P1–P5, C1–C5 with: test present? exact numbers asserted? passing?
4. Invoke the `scheduler-verifier` agent and relay its report verbatim (do not soften it).
5. If it reports blockers, fix them test-first, then re-run steps 1–4.
