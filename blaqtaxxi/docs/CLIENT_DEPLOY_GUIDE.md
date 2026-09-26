# CLIENT_DEPLOY_GUIDE.md — deploying BLAQTAXXI on your own accounts

**Status: SKELETON.** This is the outline of the guide IAS gives the client after he approves the pilot. **No step below has been verified yet.** Phase 10 produces the real guide and it is tested end to end by someone other than its author on a clean set of accounts. Do not hand this file to the client as is.

## 1. What you are getting

The same app that ran in the pilot, deployed on **your** domain and **your** accounts, with your configuration imported from the pilot. Live payments happen only here, through **your** Stripe account.

## 2. Accounts you will need (owner = you)

| Service | Purpose | Notes (all costs and limits to be verified in Phase 10) |
|---|---|---|
| Domain registrar and DNS | Your address (for example `www.blaqtaxxi.com`) | You control DNS |
| Vercel | Hosts the web app | Your team/project |
| Neon (Postgres) | Bookings and configuration | Backups and restore to be tested |
| Upstash (Redis) | Live location, caching, rate limits | |
| Google Cloud | Maps, Places, Routes | Billing account and budget alert; two restricted keys |
| Stripe | Live payments | **Your** business account; webhook |
| Resend | Email | Verified sending domain |
| SMS provider | Texting the trip link | Carrier registration may be required; lead time not verified |
| n8n Cloud (or the agreed replacement) | Reminders and notices | Credentials owned by you |
| Object storage | Car and driver photos | Assumed Vercel Blob |
| Apple Developer and Google Play (your own accounts) | The driver app. IAS never publishes it; whether and how you publish is your decision | Rules and costs not verified (U-D4) |
| Sentry, PostHog (optional) | Errors and funnel | |

## 3. Steps (outline, to be written and verified in Phase 10)

1. Create the accounts above; record who owns each.
2. Point your domain at Vercel; enable HTTPS.
3. Create the database and cache; run migrations.
4. Create Google keys with restrictions (browser key by referrer, server key by API); set a budget alert.
5. Create the Stripe live account and webhook; store keys **only** in the production environment.
6. Set environment variables from `.env.example` for `APP_ENV=production`.
7. Run `scripts/doctor`; it must pass before you continue.
8. Seed and bootstrap the admin; enable 2FA or a passkey.
9. **Import the configuration** exported from the pilot (hours, cars, prices, policy, messages, profile). Bookings and customer data are never carried over.
10. Upload real photos of your cars and your headshot.
11. Build and distribute the driver app under your Apple/Google accounts, or use the path IAS agrees with you.
12. Complete `docs/LAUNCH_CHECKLIST.md` and `docs/COMPLIANCE_CHECKLIST.md`.
13. Rehearse with a real small charge and refund, then go live.

## 4. Rollback and support

To be written in Phase 10: how to roll back a bad deploy, restore the database, disable the site, and who to call (U-H1).

## 5. Not covered here

Legal, insurance, permits, taxes, and privacy obligations. Those are yours to confirm with the right professionals; see `docs/COMPLIANCE_CHECKLIST.md`.
