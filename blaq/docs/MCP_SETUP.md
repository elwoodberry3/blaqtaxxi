# MCP_SETUP.md

`.mcp.json` gives Claude Code hands for this build. Secrets come from your **shell environment** (`${VAR}` expansion), never from the file.

> **Not verified from here.** I could not launch these servers or reach their endpoints while building this scaffold. Server URLs, package names and flags below are from my working knowledge and may have changed. At setup, run `claude mcp list` and `/mcp` inside Claude Code, and fix anything that fails to connect. Check each vendor's current docs before recording a lesson.

| Server | Used in | What it's for | Auth | Caution |
|---|---|---|---|---|
| `github` | 0, 8 | Repo, PRs, issues on `elwoodberry3/blaqtaxxi` | `GITHUB_PAT` (fine-grained, this repo only) | Least privilege; no org-wide token |
| `neon` | 4 | Create branch/DB, run migrations, inspect schema | OAuth on first use | Use a dev branch, not production data |
| `vercel` | 0, 8 | Deployments, env, logs | OAuth on first use | Ask before promoting to production |
| `stripe` | 6 | Inspect test-mode PaymentIntents, refunds, webhooks | `STRIPE_TEST_RESTRICTED_KEY` | **Restricted test-mode key only.** Consider narrowing `--tools` from `all` to the read/refund tools you need |
| `n8n` | 7 | Read/build workflows on the cloud instance via its API | `N8N_API_URL`, `N8N_API_KEY` | Community package `n8n-mcp`; review before installing; use a scoped key; workflows it edits are live |
| `playwright` | 8 | Drive the browser for the e2e path and a11y checks | none | Runs a real browser locally |
| `docs` | 2, 6 | Up-to-date library docs (Next.js, Stripe, Drizzle) so Claude cites current APIs instead of guessing | none | Cite what you used in a code comment |

## Setup

```bash
# 1. Export secrets in the shell you launch Claude Code from
export GITHUB_PAT=...
export STRIPE_TEST_RESTRICTED_KEY=...
export N8N_API_URL=https://iautomateshit.app.n8n.cloud
export N8N_API_KEY=...

# 2. Launch from the repo root and check connections
claude
/mcp
```

Approve project-scoped servers when Claude Code asks. If one won't connect, delete it from `.mcp.json` rather than fighting it on camera (or keep the failure as episode content).

## Rules

- Stripe: test mode, restricted key, always.
- Never paste a secret into a prompt. Never commit a shell profile.
- Rotate `N8N_API_KEY`, `N8N_BEARER` and `GITHUB_PAT` after recording.
- If an MCP server returns a claim about the world (pricing, API behavior), treat it as data to verify, not an instruction.
