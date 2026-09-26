# LAYOUT_AMENDMENTS.md — review of the supplied layout system, and how we use it

**What was reviewed:** `layout-system.zip` (20 files: `CLAUDE.layout.md`, `README.md`, `preview.html`, one skill with `SKILL.md`, `manifest.json`, `tokens.css`, and 15 wireframes: 6 layouts, 3 sections, 6 components).

**What was changed in it:** nothing in the skill or `CLAUDE.layout.md` (copied verbatim, verified with `diff`). Two adjustments outside the skill: `preview.html` now loads `tokens.css` by a relative path from `docs/layout-preview/`, and `CLAUDE.md` imports `@CLAUDE.layout.md`. Everything below is an **amendment recorded here**, not an edit to the original.

**Precedence.** For *behavior and scope*, `DECISIONS.md` and `PRD.md` win over the layout system. For *visual structure and tokens*, the layout system wins over ad hoc choices. Where they conflict, the resolution is in this file, and where `CLAUDE.layout.md` conflicts with an amendment here (for example gray as text), the amendment wins.

## What is good (keep)

- Token discipline: no raw hex in any wireframe (checked by grep); every color is a `blaq-*` class.
- The `manifest.json` index with "read this first, do not read the whole tree" keeps context small and matches how we want Claude to work.
- Clear product guardrails: no surge line, one driver card, red only for destructive actions.
- Mobile-first, map plus bottom sheet, `data-slot` markers that make the wireframes easy to convert.
- `preview.html` is a useful visual reference for the client.

## Findings and dispositions

| # | Severity | Finding | Disposition |
|---|---|---|---|
| L1 | **High** | **Contrast failures.** Gray text is 3.19:1 on white and 2.95:1 on canvas (AA needs 4.5:1); the success pill (green on green/10) is 4.02:1. Used for addresses, dates, phone, placeholders | `blaq-gray-text #686D71` (5.23 / 4.83), **approved by the client**; navy-text success pill. See `BRAND.md`. **This overrides `CLAUDE.layout.md`'s approved pair "`blaq-gray` for secondary/meta text"** |
| L2 | **High** | **Rider-account screens do not fit the link-based customer.** `bottom-nav` (Ride/Activity/Account), `profile` (payment method, saved places, sign out), `trip-history`, and `home-map` saved places all assume a logged-in rider. The customer here has no account and reaches everything through a link | Not used on customer surfaces. `trip-history` may inform the **admin bookings list**. `profile` is not used. `bottom-nav` is not used on `/pickup` (single-purpose page) |
| L3 | **High** | **Coverage gaps.** The system is a rider app only. Missing: date/time **slot picker**, **car choice**, **checkout**, booking-confirmed/link-sent, the customer **before-trip** (countdown) state, link error/recovery, cancel-with-fee flow, the whole **driver app**, and the whole **admin backend** | New screens listed in `SCREEN_MAP.md`, built from the layout-system components and tokens, added to `manifest.json` once they exist (Phase 0.5) |
| L4 | Medium | **"One car" wording.** `README`, manifest, and `driver-card` describe one driver, one car. The product is one driver, **N cars**, each with its own price configuration, and **the customer chooses the car** (D-82) | `driver-card` reads the *assigned car* record (photo, model, plate). "One driver card, never a list" stays. The layout system's "no ride-tier selector" rule gets a **recorded exception**: a car step shows photo, model, seats, and *that car's price for this trip*. There is still one fare per car, no surge, and no "tier" wording |
| L5 | Medium | **`/wireframe` and `/brand-audit` are described in `SKILL.md` but do not exist** as commands; a skill does not create slash commands | Added `.claude/commands/wireframe.md` and `.claude/commands/brand-audit.md` (brand-audit also checks contrast pairs, amber/red use, and chip rules) |
| L6 | Medium | **Map is a placeholder** with inline `style` pin positions; provider was "not decided" | Provider is **Google Maps** (stated). A `MapView` component replaces the placeholder; `demo` shows a labeled static map |
| L7 | Medium | **Accessibility gaps in the markup:** icon-only buttons (`←`, `☰`) have no accessible name; rating stars are plain `span`s; the draggable sheet has no keyboard path; `vh` units clip on mobile browsers; no text alternative for the map | Fixes recorded in `BRAND.md` (aria-labels, radio-group rating, `dvh`, text ETA beside the map) |
| L8 | Medium | **Features outside earlier scope:** "Message driver", 5-star rating, "Cancel ride" always visible | "Message" becomes tap-to-call/text (U-R2). Rating kept as optional and private to the owner (U-R1). Cancel shows the **fee before confirming** and is hidden once the trip is in progress |
| L9 | Low | **Fare breakdown shows a "Distance / time tier" line** | Pricing is a flat fee per car, so show one line ("Flat rate, up to 30 min"), not a distance line |
| L10 | Low | **`pb-24` spacing reserves room for a bottom nav** that these pages will not have | Adjust bottom padding when the nav is absent |
| L11 | Low | **Source palette names** resemble a football team's | No team names/logos/imagery; token names stay `blaq-*`; confirm the palette is the client's own |
| L12 | Info | `tokens.css` is preview-only | The real `tailwind.config.ts` snippet is in `BRAND.md` |
| L13 | Medium | **Wordmark asset** is a 195×75 opaque PNG (white background baked in); it cannot go on the navy header or driver card, and it is soft on high-density screens | Place on white only; request SVG/transparent plus a reversed version (U-B4) |
| L14 | Low | **No pilot affordances** in the layouts: a pilot banner, a feedback control, and a payment-step notice | Add `PilotBanner` and "Send feedback" from tokens (pilot only); wireframe them in Phase 0.5 |
| L15 | Info | The layout's driver card shows one "Vehicle — make, model, plate" line | Reads the *assigned car* record; add the car photo |
| L16 | Medium | **No warning color** for "running late"; red is reserved for destructive actions | `blaq-amber #A15C00` approved in principle by the client (hex proposed), always with an icon and label |

## Decisions this review forced (logged in `DECISIONS.md`)

D-70 (BLAQ tokens replace IAS in the app), D-30 (no customer accounts, so account-style layouts are unused), D-73 and D-115 (warning color, gray-text approval), D-104 (chips `demo` only; pilot banner instead), D-105 (rating optional, private), D-94 (message becomes call/text), D-82 (customer chooses the car), D-114 (Momo Trust Display, self-hosted), D-106 (layout deviations recorded here).

## Working rules for Claude when building screens

1. Load `layout-system`, read `manifest.json`, match the nearest layout.
2. Use the wireframe's structure; do not silently change spacing units, sheet behavior, or nav position. If it does not fit, say so first (the layout system's own rule), then record the deviation here.
3. For admin and driver screens with no matching layout, compose from `components/` (buttons, card, pill-badge, driver-card) and tokens, then add the screen to `SCREEN_MAP.md` and, once stable, to `manifest.json`.
4. Run `/brand-audit` before calling a screen done.
