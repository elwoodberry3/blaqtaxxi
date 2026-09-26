# BRAND.md — BLAQ design tokens

**Source of truth:** `CLAUDE.layout.md` (imported by `CLAUDE.md`) and `.claude/skills/layout-system/`. This supersedes the earlier IAS-token version of this file: BLAQTAXXI is a client product and uses the client's palette. IAS branding appears only on course material, not in the app.

Contrast ratios below were **computed** (WCAG 2.x relative luminance), not estimated. AA needs 4.5:1 for normal text, 3:1 for large text and UI graphics.

## Tokens (Tailwind `theme.extend.colors`)

| Token | Hex | Role |
|---|---|---|
| `blaq-navy` | `#002244` | Primary: headers, driver card, headings |
| `blaq-royal` | `#0072CE` | Accent: primary CTA, links, active pin, filled stars |
| `blaq-green` | `#008752` | Success/confirm: trip confirmed, online |
| `blaq-red` | `#862633` | Danger: cancel, no-show, sign out, cancelled. **Destructive/negative only** |
| `blaq-gray` | `#8A9197` | Dividers, disabled, borders, decorative. **Not for text (see below)** |
| `blaq-black` | `#000000` | Body text, icons |
| `blaq-canvas` | `#F5F6F7` | Page background. *Assumed by the layout system, not confirmed (U-B1)* |
| `blaq-white` | `#FFFFFF` | Cards, sheets, reversed text |
| `blaq-gray-text` | `#686D71` | **Proposed** text gray: a darker shade of `blaq-gray`. Needs client approval (U-B1) |

```ts
// tailwind.config.ts (extend)
colors: {
  'blaq-navy': '#002244', 'blaq-royal': '#0072CE', 'blaq-green': '#008752',
  'blaq-red': '#862633', 'blaq-gray': '#8A9197', 'blaq-gray-text': '#686D71',
  'blaq-black': '#000000', 'blaq-canvas': '#F5F6F7', 'blaq-white': '#FFFFFF',
}
```

Use `100dvh`/`dvh` units rather than `vh` for map-plus-sheet layouts so mobile browser chrome does not clip the sheet.

## Measured contrast (as used in the wireframes)

| Pair | Ratio | Result |
|---|---|---|
| White on navy | 16.00 | Pass |
| White on royal (primary CTA) | 4.89 | Pass |
| White on green | 4.58 | Pass (narrow) |
| White on red | 8.98 | Pass |
| Black on canvas | 19.41 | Pass |
| Navy on canvas | 14.79 | Pass |
| Royal text on white | 4.89 | Pass |
| Royal text on canvas | 4.52 | Pass (narrow; do not go smaller or lighter) |
| Red text on white | 8.98 | Pass |
| **Gray text on white** | **3.19** | **FAIL for text** (fine for 3:1 UI graphics like unfilled stars) |
| **Gray text on canvas** | **2.95** | **FAIL** even for graphics |
| **Green text on green/10 tint (success pill)** | **4.02** | **FAIL** |
| Red text on red/10 tint (danger pill) | 7.55 | Pass |
| Navy text on gray/15 tint (neutral pill) | 13.78 | Pass |
| `blaq-gray-text` `#686D71` on white / canvas | 5.23 / 4.83 | Pass |

### What fails in the supplied wireframes, and the fix

| Where | Problem | Fix (no new brand color needed except the gray shade) |
|---|---|---|
| Addresses, dates, phone, placeholder text (`text-blaq-gray text-xs`) in home-map, trip-history, profile | 3.19 / 2.95 | Use `text-blaq-gray-text` (pending approval) for all gray **text**; keep `blaq-gray` for borders/dividers/disabled |
| Success pill: `text-blaq-green` on `bg-blaq-green/10` | 4.02 | `bg-blaq-green/10 text-blaq-navy` with a green dot, or solid `bg-blaq-green text-blaq-white` (4.58) |
| Map placeholder text | 2.95 | Real map replaces it; keep placeholder text at `blaq-gray-text` |

## Status language (customer, driver, admin)

| State | Treatment |
|---|---|
| Confirmed / paid | Success pill (navy text, green dot) |
| Countdown (before T-30) | Navy on white card; time in a tabular-figures style |
| Live (T-30 → arrival) | Driver card (navy) + royal pin + "Driver is 12 min away" |
| During trip | Route line and car marker in royal; ETA in navy |
| Complete | Green check, receipt |
| Cancelled / no-show | Danger pill (red) |
| **Running late / warning** | **No token exists.** Red is reserved for destructive actions, so do not use it for lateness. Until the client approves a warning color (U-B1): navy text + warning icon + explicit label ("Running about 5 min late"). Color is never the only carrier of meaning |
| Demo/staging chips | Neutral gray pill. **Never rendered in production** |

Color ratio: `blaq-navy` and `blaq-royal` together stay under about 30% of any screen; canvas and white are the majority surface.

## Typography

System font stack from the layout system unless the client supplies a brand font (U-B3). Times, ETAs, and prices use tabular figures so digits do not jump as the countdown ticks. Minimum 16 px body on the customer and driver surfaces; the wireframes' `text-xs` (12 px) is for metadata only, at `blaq-gray-text` or darker.

## Maps

Provider: **Google Maps** (stated). Map styling stays neutral so the royal route line and the navy/royal pins carry the signal. The wireframes' inline `style="top:40%; left:55%"` pins are placeholders and are replaced by real map markers. Demo mode renders a static placeholder map, labeled as demo.

## Naming and provenance

The source palette's color names appear to come from a football team's palette. The hex values are just colors, but do not use the team's name, logo, or imagery in the app, the repo copy, or the course, and confirm the palette is the client's own intended brand (U-B1). Token names stay `blaq-*`.

## Voice

Direct, clear, short. Customer copy: "Driver is 12 min away." "Pickup in 32 h 10 m." "Check back 30 minutes before your pickup to see where your driver is." No corporate polish, no buzzwords, no invented reviews or ratings.

## Accessibility (measured requirements)

- WCAG AA on every pair above; icon-only buttons need `aria-label`; the map region needs a text alternative and a text ETA beside it.
- Focus rings on all interactive elements (royal ring on light, white ring on navy).
- ETA changes announced with `aria-live="polite"`, not on every tick.
- Rating is an accessible radio group, not styled `span`s.
- Touch targets at least 44 px on `/drive` and `/admin`.
