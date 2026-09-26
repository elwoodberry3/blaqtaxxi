<!--
  BLAQ (Blaqtaxxi) — layout-system rules block.
  Import this from your project's CLAUDE.md with:  @CLAUDE.layout.md
  Do not overwrite an existing CLAUDE.md — append the import line instead.
  This file is read on every session start. The wireframe library itself
  is read on demand, indexed by .claude/skills/layout-system/manifest.json.
-->

# BLAQ Layout System

BLAQ (Blaqtaxxi) is a one-driver, one-car, one-fare-at-a-time ride-booking
app. UI direction: a Lyft-like consumer ride app — map-first screens,
bottom-sheet flows, pill buttons, rounded cards, a single bottom nav.

## Design tokens — Tailwind, not raw hex

Every class in this project MUST resolve to one of the tokens below via
`tailwind.config` (`theme.extend.colors`). Never hardcode a hex value or an
arbitrary Tailwind value (`bg-[#0072CE]`) in a component — always use the
token name, so a future re-skin is a one-file change.

| Token           | Hex       | Source name             | Usage                                  |
|-----------------|-----------|--------------------------|-----------------------------------------|
| `blaq-navy`     | `#002244` | Cowboys Blue / Navy      | Primary — nav, headers, driven text     |
| `blaq-royal`    | `#0072CE` | Royal Blue               | Brand accent — primary CTA, active map pin, links |
| `blaq-green`    | `#008752` | Victory Green            | Success / confirm — trip confirmed, fare locked, online status |
| `blaq-red`      | `#862633` | Republic Red             | Danger — cancel, no-show, destructive actions only |
| `blaq-gray`     | `#8A9197` | Silver / Gridiron Gray   | Muted text, dividers, disabled state    |
| `blaq-black`    | `#000000` | Stealth Black            | Body text on light surfaces, icons      |
| `blaq-canvas`   | `#F5F6F7` | (assumed — not in source palette, flagged for confirmation) | Page background, card fill |
| `blaq-white`    | `#FFFFFF` | (assumed — standard)     | Card surfaces, reversed text on dark    |

**Assumption flagged under Article IX (AI Governance):** the brand guide
supplied five brand colors and black. It did not specify a canvas/background
neutral. `blaq-canvas` and `blaq-white` are a reasonable default for a
Lyft-like light UI, not a confirmed brand value — swap them in
`tailwind.config` if a canvas color is specified later.

Color ratio: keep `blaq-navy` / `blaq-royal` combined under ~30% of any
screen. `blaq-canvas` / `blaq-white` should read as the majority surface, in
line with the 60/30/10 pattern used across IAS builds.

Approved pairs:
- White text on `blaq-navy` (headers, primary buttons)
- `blaq-black` body text on `blaq-canvas` / `blaq-white`
- White text on `blaq-royal` (primary CTA)
- White text on `blaq-green` (confirmation states)
- White text on `blaq-red` (destructive confirm only, never decorative)
- `blaq-gray` for secondary/meta text only — never for body copy or CTAs

## Operational rules

1. BEFORE generating any new page or screen, read
   `.claude/skills/layout-system/manifest.json` first — it indexes every
   layout, section, and component with its purpose and when to use it.
   Do not read the whole `wireframes/` tree up front.
2. Match the requested screen to the nearest layout in the manifest. Do not
   invent a new page structure when an existing layout fits.
3. Use a wireframe's structure as the baseline, then populate it with real
   content and logic. Do not silently alter its structure (spacing units,
   bottom-sheet behavior, nav position) — if a layout genuinely doesn't fit,
   say so before deviating.
4. Every class used must be a `blaq-*` token or a standard Tailwind utility
   (spacing, layout, typography scale). No hardcoded hex, no ad-hoc colors.
5. Stack: Next.js (App Router), TypeScript, Tailwind CSS, Vercel, n8n Cloud
   for orchestration — per the IAS Baseline Stack (Article VIII). Do not
   introduce a competing UI or styling library.
