---
name: layout-system
description: BLAQ (Blaqtaxxi) wireframe library — a one-driver, one-fare ride-booking app. Load whenever generating or reviewing a page, screen, or component for this project.
---

# Brand & Wireframe System Skill

## Purpose

Ensures every screen generated for BLAQ (Blaqtaxxi) conforms to the brand
color tokens and the ride-app layout structure defined in this library,
instead of Claude inventing a new layout or a new color per request.

## Available slash commands

- `/wireframe <layout_or_component_name>` — inject a template from
  `wireframes/` into the active workspace (e.g. `/wireframe home-map`,
  `/wireframe driver-card`).
- `/brand-audit` — audit the current active HTML/CSS/TSX against the token
  table in `../../../CLAUDE.layout.md`, flagging any hardcoded hex value or
  off-token color use.

## Operational constraints

1. Before generating any new UI code or page, read `manifest.json` in this
   directory. It indexes every layout, section, and component with its
   purpose and when to use it. Read the full `wireframes/` tree only if the
   manifest doesn't resolve the request.
2. Match the user's intent to the nearest entry in the manifest (e.g. a
   request for a "trip receipt screen" maps to `fare-summary`). Do not
   invent an unrelated layout when an existing one fits.
3. Read the matched wireframe file directly from `wireframes/` and use its
   structure — sections, data-slot placeholders, component composition — as
   the skeleton. Populate it with real content, real data bindings, and
   real business logic. Convert the static HTML into the project's actual
   stack (Next.js + TypeScript + Tailwind) rather than shipping raw HTML.
4. Apply only the color tokens defined in `CLAUDE.layout.md`
   (`blaq-navy`, `blaq-royal`, `blaq-green`, `blaq-red`, `blaq-gray`,
   `blaq-black`, `blaq-canvas`, `blaq-white`). Never hardcode a hex value
   or introduce a color outside this table.
5. This is a one-driver, one-fare app — never render a driver list, a
   ride-tier selector, or a surge/dynamic-pricing element. If a request
   implies one of these, flag the mismatch before building it.
6. `blaq-red` is reserved for destructive or negative-status actions
   (cancel, no-show, sign out, cancelled status) — never a default or
   decorative color.
