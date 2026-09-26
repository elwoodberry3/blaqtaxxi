---
description: Pull a layout-system wireframe into the workspace and convert it (usage /wireframe home-map)
argument-hint: <layout, section, or component name>
---

Load the `layout-system` skill. Then:

1. Read `.claude/skills/layout-system/manifest.json` only. Match **$ARGUMENTS** to the nearest entry (by `name`, then `when_to_use`). If nothing matches, say so, check `docs/SCREEN_MAP.md` for a planned-new screen, and tell me before inventing a structure.
2. Read that one wireframe file and any file listed in its `composed_from`. Do not read the rest of `wireframes/`.
3. Read `docs/LAYOUT_AMENDMENTS.md` and apply the recorded dispositions for that layout (for example: no saved places on home-map, no bottom nav on `/pickup`, gray text uses `blaq-gray-text`, icon buttons get `aria-label`).
4. Convert the static HTML into a real Next.js + TypeScript + Tailwind component in the route named in `docs/SCREEN_MAP.md`, using only `blaq-*` tokens. Keep the wireframe's structure and `data-slot` names; do not silently change spacing units or sheet behavior.
5. Wire it to real data shapes (or typed demo fixtures on `demo`). Add loading, empty, and error states.
6. Run `/brand-audit` on what you built and show the result.
