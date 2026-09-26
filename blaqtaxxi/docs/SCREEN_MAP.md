# SCREEN_MAP.md — every screen, by audience

**Layout source:** ✔ *exists* in the layout system · ◐ *adapt* an existing layout · ✚ *new* (compose from layout-system components and `blaq-*` tokens, then add to `manifest.json`).
Phase = when it is built (`BUILD_PLAN.md`). "0.5" means wireframed first, then built in the named phase.

## A. Customer (`/book`, `/pickup/[link]`) — no account

| ID | Screen | Layout | Key content and behavior | Phase |
|---|---|---|---|---|
| C1 | Start a booking | ◐ `home-map` | Pickup and drop-off (Google Places, DFW-bounded), date, party size. **Remove** saved places and menu. Out-of-area → contact prompt | 3 |
| C2 | Pick a time | ✚ | Dynamic slots from `findSlots()`, "tight fit" badge, "why is 9:00 gone?" explanation, "Ride now" if feasible. Slot hold timer | 3 |
| C3 | Confirm and price | ◐ `ride-confirm` | Route preview, car shown (from assigned car), **one** flat fare, cancellation terms in plain words, contact fields (name, phone, email). No tier list | 3 |
| C4 | Pay | ✚ | Stripe payment element, wallets. Price locked, hold countdown | 6 |
| C5 | Booked | ✚ | Confirmation, the **link** shown on screen and sent by message, add-to-calendar | 6 |
| C6 | **Before the trip** | ◐ `active-trip` (pre-trip variant) | Trip details (pickup, drop-off, time, car photo/model/plate, amount paid), **countdown** ("in 32 h 10 m"), "Check back 30 minutes before your trip to see where your driver is", cancel with fee shown | 5 |
| C7 | **Driver on the way** (from T-30) | ✔ `active-trip` | Map with driver marker, `driver-card`, ETA, call/text, live-status pill, "last seen N s ago" when stale | 5 |
| C8 | **During the trip** | ◐ `active-trip` (in-trip variant) | Google Map with the car on the route, drop-off pin, **estimated arrival**. No cancel button | 5 |
| C9 | **After the trip** | ◐ `fare-summary` | Trip log (times, route, car), completion **receipt** (flat-fee line, total, payment method last 4), optional private rating | 6/8 |
| C10 | Link problems | ✚ | Not found / expired / recover ("find my ride": last name + last 4 + one-time code) / rate-limited | 3 |
| C11 | Cancel | ✚ | Shows fee before confirming; refund amount; result | 6 |
| C12 | Cancelled / no-show / late notice states | ◐ pill-badge + card | Clear status, refund info, what happens next | 6 |

Derived customer phase (not stored): `before` → `approaching` (T-30 to arrival) → `in_trip` → `after`, plus `cancelled`, `no_show`, `expired`.

## B. Driver console (`/drive`) — the "Lyft driver" view

| ID | Screen | Layout | Key content and behavior | Phase |
|---|---|---|---|---|
| D1 | Today | ✚ | Timeline of today's rides and blocks, gaps and drive minutes (from the engine), next pickup card | 4 |
| D2 | Next pickup | ✚ | Customer first name and last initial, pickup address, time, **time to spare**, late-risk flag, big **Navigate** button | 4/5 |
| D3 | Heading to pickup | ✚ | **Navigate** → Google Maps deep link (`dir/?api=1&destination=…&dir_action=navigate`, verified format), ETA, big **Arrived** button | 5 |
| D4 | At pickup | ✚ | Wait timer (5 min), call/text, verify customer (ask last name), **Rider in car**, **No-show** (enabled after the wait) | 5 |
| D5 | **In trip: where to go** | ✚ | On "Rider in car" the primary action becomes **Navigate to drop-off** (deep link), route ETA, **Complete** | 5 |
| D6 | Between rides | ✚ | Next pickup, time to spare, deadhead estimate, cascade warnings if late | 5 |
| D7 | Ride detail | ✚ | Full details for one booking | 4 |
| D8 | Offline / no GPS / permission denied | ✚ | Visible state; what the customer sees meanwhile | 5 |

## C. Admin backend (`/admin`) — secure, configures what customers see

| ID | Screen | Layout | Controls | Phase |
|---|---|---|---|---|
| A1 | Sign in | ✚ | Method per U-A1; 2FA/passkey | 4 |
| A2 | Dashboard | ✚ | Today, upcoming, alerts (late risk, failed payments, config warnings) | 4 |
| A3 | Calendar | ✚ | Day/week: bookings, blocks, overrides. Add block, move day | 4 |
| A4 | Availability | ✚ | Default hours (05:00–24:00), **per-date overrides** (shorten, move, close), shows bookings past a new cutoff before saving | 4 |
| A5 | Cars | ✚ | List/detail: photo, year/make/model/color/plate, seats, active flag, **which car is assigned when** | 4 |
| A6 | Prices | ✚ | Per-car price config (tiers), **versioned** with effective date, quote preview | 4 |
| A7 | Policy | ✚ | Cancellation and no-show amounts, wait time; versioned | 6 |
| A8 | Operations | ✚ | Service area, lead time, horizon, buffers, dwell times, ride-now on/off | 4 |
| A9 | Bookings | ◐ `trip-history` list | Search, filter, detail, cancel/refund with confirmation | 6 |
| A10 | Messages | ✚ | Copy for confirmation, T-30 reminder, late notice, no-show, receipt; preview | 7 |
| A11 | Business profile | ✚ | Name, phone, email, logo, receipt fields, driver display name/photo and contact number | 6 |
| A12 | Audit log | ✚ | Who changed what config, when (append-only) | 4 |
| A13 | Account and security | ✚ | Sessions, 2FA/passkey | 4 |

## D. Not built (from the supplied layouts)

`profile`, `bottom-nav`, saved places, payment-method settings, sign-out for customers: they assume a rider account (`LAYOUT_AMENDMENTS.md` L2).

## E. Cross-cutting states every screen must handle

Loading, empty, error, offline, session expired (admin/driver), slot just taken, payment failed, location stale, out of area. Each has a visible, plain-language state, and none of them shows a TodoChip in production.
