# BRAND.md — design tokens (IAS system, swappable)

Source: `ias_color_system.csv` and the IAS brand rules. BLAQTAXXI is built as an **IAS demo** (D-70). A `brand.config.ts` seam lets the operator's own brand replace these tokens without touching components.

> **Rule:** color is a splash, not a flood. **60 / 30 / 10** — 60% canvas, 30% brand, 10% accent.

## Core tokens

| Token | Hex | Use |
|---|---|---|
| Deep Slate Teal `--ias-primary` | `#0A2E36` | Headers, nav, dark surfaces, CTA backgrounds (30%) |
| Muted Seafoam `--ias-secondary` | `#3F7266` | Icons, dividers, secondary labels |
| Kinetic Emerald `--ias-accent` | `#00E5A3` | CTAs, links, active states only (10%). **One use per view. Dark text only** |
| Ink Blue-Gray `--ias-dark` | `#111827` | All body type, strong borders |
| Pure Ash `--ias-light` | `#F9FAFB` | Page canvas, section fills, code blocks (60%) |
| White `--ias-white` | `#FFFFFF` | Card fills, reversed type on dark |
| Muted Gray `--ias-muted` | `#6B7280` | Captions, eyebrows, metadata |
| Border Gray `--ias-mid` | `#E5E7EB` | Hairlines, dividers, input strokes |

## Tints and shades

| Group | Tokens |
|---|---|
| Primary | 50 `#E8F0F1` · 100 `#C1D5D9` · 400 `#4B7A8A` · 800 `#062028` · 950 `#030F12` |
| Secondary | 50 `#E6F0EE` · 200 `#9BBEC0` · 400 `#5C8A8C` · 700 `#2A5047` · 900 `#1A3330` |
| Accent | 600 `#00B882` (pressed CTA) · `rgba(0,229,163,0.04/0.05/0.15/0.30)` for selection, drag, ring, stroke |
| Neutral | Disabled `#9CA3AF` · Body Gray `#374151` |

## Approved foreground/background pairs (use only these)

| Pair | FG on BG | Use |
|---|---|---|
| White on Primary | `#FFFFFF` / `#0A2E36` | Headers, nav, footers |
| Emerald on Primary | `#00E5A3` / `#0A2E36` | CTAs on dark surfaces |
| Primary on White | `#0A2E36` / `#FFFFFF` | Body, cards |
| Primary on Emerald | `#0A2E36` / `#00E5A3` | CTA buttons, badges |
| White on Seafoam | `#FFFFFF` / `#3F7266` | Icon fills, state indicators |
| Seafoam on Tint | `#3F7266` / `#E6F0EE` | Pills, tags |
| Ink on Ash | `#111827` / `#F9FAFB` | Code blocks, section fills |
| Reversed Secondary | `#9BBEC0` / `#0A2E36` | Wordmark accent on dark |

Never put Emerald text on white or Ash (it fails contrast). Emerald is a *fill* with Primary text, or text on Primary.

## Type

Space Grotesk (UI, headings) and Space Mono (times, ETAs, IDs, code). Load via a **runtime `<link>`**, with system fallbacks (`ui-sans-serif`, `ui-monospace`). Not `next/font/google` (see `CLAUDE.md` gotcha #3).

Times and ETAs use tabular figures in Space Mono so digits don't jump as the countdown ticks.

## CSS variable seam

```css
:root{
  --ias-primary:#0A2E36; --ias-secondary:#3F7266; --ias-accent:#00E5A3;
  --ias-dark:#111827; --ias-light:#F9FAFB; --ias-white:#FFFFFF;
  --ias-muted:#6B7280; --ias-mid:#E5E7EB;
}
```

Tailwind maps semantic names (`bg-brand`, `text-ink`, `bg-canvas`, `accent`) to these variables, so a future `brand.config.ts` only swaps the variable block.

```ts
// brand.config.ts (target shape)
export const brand = {
  name: 'BLAQTAXXI',                 // display name; IAS demo skin by default
  tokens: { primary:'#0A2E36', secondary:'#3F7266', accent:'#00E5A3', dark:'#111827', light:'#F9FAFB' },
  logo: null,                        // TodoChip until provided
};
```

## Status language (product states → tokens)

| State | Treatment |
|---|---|
| Confirmed | Seafoam on Tint pill |
| Countdown (before T-30) | Ink on Ash card, Space Mono time |
| **Live** (T-30 → pickup) | The single Emerald use in the view: Primary-on-Emerald "Live" badge |
| In progress | White on Seafoam |
| Complete | Primary on White, receipt |
| Late risk / error | **See gap below** |

### Gap: the IAS palette has no warning or error color

`ias_color_system.csv` defines no red/amber semantic tokens. Late-risk and error states need one. **Until Steve approves a warning/danger token pair**, use: Ink text, a leading icon, a text label ("Running 5 min late"), and a Primary-800 background. Do not invent a red. Logged as `D-73` in `DECISIONS.md`. Never rely on color alone to carry meaning.

## Voice

Direct, clear, technical, honest, demonstrable. "Here's what I built," not "I'm passionate about." No corporate polish, no buzzwords, no fabricated ratings or testimonials. Rider-facing copy is plain and short: *"Driver is 12 min away."* *"Pickup in 32 h 10 m."*

## Accessibility

WCAG AA contrast on every approved pair. Focus rings on all interactive elements (Emerald ring on dark, Primary ring on light). ETA changes announced via `aria-live="polite"`. Minimum 44 px touch targets on the driver console (he'll be tapping while parked, in daylight).
