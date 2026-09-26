# BLAQ (Blaqtaxxi) — layout-system

A drop-in wireframe library and Claude Code skill for the BLAQ ride-booking
app. Generated for a one-driver, one-car, one-fare-at-a-time app, using the
brand palette below and a Lyft-like UI direction.

## What's in here

```
layout-system/
├── README.md                 # this file
├── CLAUDE.layout.md          # token table + operational rules — import this into CLAUDE.md
├── preview.html              # visual gallery of all 6 layouts (open directly in a browser)
└── .claude/skills/layout-system/
    ├── SKILL.md               # skill instructions + slash commands
    ├── manifest.json          # index of every layout, section, component
    └── wireframes/
        ├── tokens.css         # preview-only token reference (mirror in tailwind.config)
        ├── layouts/           # 6 full-page skeletons
        ├── sections/          # 3 reusable page fragments
        └── components/        # 6 reusable UI pieces
```

## Install

1. Copy the `.claude/skills/layout-system/` folder into your project's
   `.claude/skills/` directory.
2. Copy `CLAUDE.layout.md` into your project root.
3. If your project already has a `CLAUDE.md`, **do not overwrite it** — add
   one line near the top:
   ```
   @CLAUDE.layout.md
   ```
   Confirm your Claude Code version supports `@`-imports before relying on
   this; if it doesn't, paste the contents of `CLAUDE.layout.md` directly
   into `CLAUDE.md` under its own heading instead.
4. In `tailwind.config`, register the token names under
   `theme.extend.colors` using the hex values in `CLAUDE.layout.md`
   (`blaq-navy`, `blaq-royal`, `blaq-green`, `blaq-red`, `blaq-gray`,
   `blaq-black`, `blaq-canvas`, `blaq-white`) so `bg-blaq-royal` etc. resolve
   to real Tailwind utilities.
5. Open `preview.html` in a browser to see all 6 layouts before building.

## Using it

Ask Claude Code to build a screen by name or by intent — e.g. "build the
ride confirmation screen" or "build the receipt page." It should:

1. Read `manifest.json` to find the nearest matching layout.
2. Read that layout's wireframe file for structure.
3. Rebuild it as a real Next.js/TypeScript/Tailwind component, using only
   the `blaq-*` tokens.

Use `/wireframe <name>` to pull a specific file in directly, or
`/brand-audit` to check existing code against the token table.

## Two open items — confirm before shipping

- **Canvas/background color.** The five-color brand guide didn't include a
  neutral page background. `blaq-canvas` (`#F5F6F7`) and `blaq-white`
  (`#FFFFFF`) in this library are a working assumption for a light,
  Lyft-like UI — not a confirmed brand value.
- **Map provider.** Every map area in these wireframes is a placeholder
  block. Google Maps vs. Mapbox vs. something else isn't decided here.

## Brand palette (source)

| Name | Hex |
|---|---|
| Cowboys Blue / Navy | `#002244` |
| Royal Blue | `#0072CE` |
| Victory Green | `#008752` |
| Republic Red | `#862633` |
| Silver / Gridiron Gray | `#8A9197` |
| Stealth Black | `#000000` |
