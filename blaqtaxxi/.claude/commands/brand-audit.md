---
description: Audit app code against the BLAQ tokens, contrast rules, and chip rules
---

Audit `app/`, `components/`, and any CSS for brand and accessibility violations. Show the commands you ran and their output; then a findings list (file:line, rule, fix). Do not edit anything unless I ask.

1. **No raw colors.** Must return nothing:
   `grep -rnE "#[0-9A-Fa-f]{3,8}\b|rgba?\(|bg-\[|text-\[#|border-\[#" app components --include=*.tsx --include=*.ts --include=*.css`
   (tokens.css and `tailwind.config.ts` are allowed to define hex; nothing else is.)
2. **Only `blaq-*` color classes.** Flag any Tailwind palette color (`text-gray-500`, `bg-blue-600`, etc.).
3. **Gray is not text.** Flag `text-blaq-gray` on text elements (allowed on borders, dividers, disabled backgrounds, unfilled stars). Text uses `text-blaq-gray-text` or darker.
4. **Red is destructive only.** Flag `blaq-red` used for anything except cancel, no-show, cancelled status, or sign-out.
5. **Approved pairs and contrast.** Check foreground/background combinations against the table in `docs/BRAND.md`; flag any pair under 4.5:1 (text) or 3:1 (UI graphics), including tinted-pill text.
6. **Accessibility basics.** Icon-only buttons need `aria-label`; ratings are a radio group; ETA/status regions use `aria-live="polite"`; map has a text alternative; touch targets on `/drive` and `/admin` are at least 44 px.
7. **Chips.** `TodoChip`/`StatusChip` must be gated by `NEXT_PUBLIC_SHOW_TODO_CHIPS`; a production build must render none. Run `NEXT_PUBLIC_SHOW_TODO_CHIPS=false npm run build` and grep the output for chip markers.
8. **Product guardrails.** No driver list, no ride-tier selector (unless the U-V2 flag is on), no surge or dynamic-pricing wording, no customer account/profile screens.
