---
name: n8n-async-layer
description: Use when creating or changing n8n workflows, outbound webhooks to n8n, the /api/n8n/* endpoints, notifications (confirmation, T-30 reminder, late notice, no-show, Slack digest), or notification idempotency.
---

# n8n-async-layer

n8n Cloud: `https://iautomateshit.app.n8n.cloud`. It owns the **async layer only**. If a user is waiting on the response, it does not belong here.

## What belongs in n8n

Booking-confirmed message · T-30 reminder · late-risk notice · no-show follow-up · daily Slack digest · optional HubSpot sync. Nothing else.

## What never goes in n8n

Slot search, feasibility, booking insert, location pings, ETA, payment authorization. (Real-time vs async split: `CLAUDE.md` §3.2.)

## Contract with the app

Outbound (app → n8n): `lib/notify/dispatch.ts` POSTs JSON to `N8N_WEBHOOK_BASE/<workflow>` with header `Authorization: Bearer ${N8N_BEARER}`. Never block the user's request on it: fire after the transaction commits; on failure write the event to `notifications` with `status='pending'` for retry.

Inbound (n8n → app):
- `GET /api/n8n/due` returns notifications due now (T-30 reminders, unsent late notices).
- `POST /api/n8n/ack` marks a notification sent. **Idempotent** on `idempotency_key`.
- Both require `Authorization: Bearer <secret>`; wrong or missing → `401`.

## Auth gotchas (these cost hours before)

- Header key must be exactly `Authorization`. Value must include the `Bearer ` prefix (with the space).
- **Credential ownership vs project-level sharing** are different in n8n Cloud; a credential shared to the project but owned by another user can fail silently. When a workflow returns 401 with a correct secret, check ownership and sharing first.
- Use a separate secret per environment; rotate after recording a demo.

## Idempotency

`notifications.idempotency_key` = `${bookingId}:${template}:${scheduledFor}`. Unique index. Duplicate delivery from n8n retries must send once. Test it.

## Workflow conventions

- One workflow per JSON file in `n8n/workflows/`, named `blaqtaxxi-<purpose>.json`.
- Every workflow starts with a Webhook or Schedule trigger, an early auth check, and an explicit error branch that posts a Slack alert. No silent failures.
- Schedule triggers: default every 5 minutes for `due` polling. Verify the n8n plan's execution limits before choosing a shorter interval.
- Keep message copy in one Set node so it's easy to review. Plain, short, direct: "Driver is 12 min away."
- Strip secrets from exported JSON before committing.

## Demo mode

If `N8N_WEBHOOK_BASE` is unset, `dispatch` writes to an in-memory outbox and `/dev/outbox` renders it. The whole flow must be demoable without n8n.

## Tests

Wrong Bearer → 401 · duplicate `ack` → single send · outbound failure leaves a `pending` row · no route handler awaits n8n on the user's critical path.

## Free-tier / integration notes

- HubSpot Free allows only 10 custom contact properties. If CRM sync is added, triage properties first; lowest-fill go first.
- SMS needs carrier registration in the US (lead time). Use email + web push until then; SMS stays behind the seam.
