---
name: admin-backend
description: Use when building or changing the /admin backend and anything it configures: availability and date overrides, blocks, cars, per-car price configs, policy amounts, service area, notification copy, business profile, audit log, roles and admin auth.
---

# admin-backend

The owner-driver logs into a secure backend that configures **what customers see and what the engine allows**. Read `docs/SCREEN_MAP.md` §C, `docs/ARCHITECTURE.md` §3, and `docs/DECISIONS.md` (sections A, E, I, K).

## Principles

1. **Config drives the customer.** Hours, overrides, cars, prices, policy amounts, copy: all come from admin-managed data, never constants in components.
2. **Versioned, never mutated.** Price configs and policies are append-only versions with an `effectiveFrom`. A booking snapshots its `priceConfigVersion`, amount, and `policyVersion`. Editing config never rewrites an existing booking. Test this.
3. **Audit everything.** Every config write appends to `config_audit_log` (actor, entity, before, after, time). The log is append-only and read-only in the UI.
4. **Authorize on the server.** Every `/api/admin/*` handler checks the session and role (`admin`), regardless of what the UI hides. Auth.js v5; method and strength per `U-A1` (2FA/passkey required before production). Rate-limit sign-in.
5. **Preview before commit.** Price and policy forms show a live preview ("a 38-minute trip costs $25.00") and state plainly that existing bookings are unaffected.
6. **Conflicts are explicit.** Shortening or closing a day that already has bookings lists them and requires a choice; never auto-cancel (D-45). Assigning a car to overlapping windows is rejected. Cancelling a paid booking shows the refund amount first.
7. **Two-device safety.** Use optimistic concurrency (a `version` column) so an edit from a stale tab fails visibly instead of overwriting.
8. **Destructive actions** use the red destructive button and a confirmation that names what will happen. Red only for destructive actions.

## Cars and price configs

- A car has: label, year/make/model/color, plate, seats, photo, active flag, `priceConfigId`. No plate or photo in the repo; they live in the database.
- The **scheduling resource is the driver**, not the car (D-83). `vehicle_assignments` windows say **when each car is offered** (the scheduler returns `CAR_NOT_ASSIGNED` outside them). **Reject overlapping windows.** "Switch car" mid-day creates a 30-minute **swap block** at the base (D-84); never model swaps any other way.
- Price config = ordered tiers `{upToMinutes | null, cents}` evaluated against the **off-peak reference trip time**; first tier with `upToMinutes >= minutes` wins; `null` is unbounded. Money is integer cents. Fixture: `docs/fixtures/vehicles.json`.
- The **customer chooses the car** (D-82): the booking flow offers only cars assigned at that time, with a slot, and enough seats; if only one car is offered the choice is hidden. Cars flagged `is_example` show an "example" label in the pilot admin.

## Media, export/import, and the pilot

- **Media:** car, driver, and logo images are uploaded through the admin to object storage (assumed Vercel Blob, D-113) and referenced by URL; validate type and size; never commit client images. Crop/resize on upload so the driver card and car cards render consistently.
- **Config export/import (A14):** exports hours, overrides, cars, price configs, policy, templates, business profile, and media references as versioned JSON, **never bookings or customer data**; import validates the schema and shows a diff before applying and writes to the audit log. This is how the pilot's tuned config moves to the client's own deployment (D-116).
- **Pilot feedback inbox (A15):** shows feedback submitted from any screen (role, screen, environment, note) and lets the owner mark it triaged.

## Availability

- Default day (05:00–24:00 America/Chicago) plus per-date overrides `{date, start?, end?, closed?}`. The end time is the last time a ride may **start** (D-09). Effective window = override if present, else default. Use `lib/time.ts`; test DST days.

## Build order per screen

Type and Zod schema → server action/route with authz and audit → test (including the authz-denied case and the "existing booking unchanged" case) → screen from `components/` and `blaq-*` tokens → `/brand-audit`.

## Accessibility and UX

44 px targets, visible focus, labeled inputs, inline validation, `aria-live` for save results, works on a phone (he may edit from the car). No customer data in URLs or logs.

## Never

- Hardcode a price, hour, policy amount, or customer-facing string.
- Expose admin data (other customers' names, phones, addresses) to any non-admin route.
- Display TodoChips in a production build.
