# UNKNOWNS.md — what is still open, and what it blocks

Updated 2026-09-25 after the layout-system review and the production/multi-car/three-surface changes.

**How to read this.** Nothing here stops you from firing the Episode 0 prompt (plan mode is read-only). But six answers will change what Claude *plans*, so answer those first (Section 1). Everything else is either needed by a later phase (Section 2) or has a safe default that Claude will build on and log (Section 3).

**Rule for Claude:** never assume an item marked **BLOCKING**. Stop and ask (`AskUserQuestion`). Defaulted items are built on the default and logged in `DECISIONS.md` as `assumed`.

Legend: **Answerer** = who can settle it. *Steve* = you. *Client* = the owner-driver of BLAQTAXXI. *Spike* = settled by a test, not an opinion.

---

## 1. Answer before firing the first prompt (they change the plan)

| ID | Question | Why it matters | Blocks | Default if unanswered | Answerer |
|---|---|---|---|---|---|
| **U-D1** | **What phone does the driver use (iPhone or Android), and is a thin native app shell acceptable, or must the driver side be a pure web app (PWA)?** | The driver needs turn-by-turn in Google Maps while our app keeps sending his location so the customer's live map moves. *Verified:* Google's Navigation SDK is native-only (Android/iOS, Flutter, React Native), not web; Maps URLs launch navigation with no key. *My understanding, unverified on his phone:* a PWA sent to the background stops reporting location, so the customer's map would freeze mid-drive. This is the highest-risk unknown | Phase 5 (live tracking). Phase 0.5 spike settles it | **No default.** Run spike SP-1 first; if it fails, a Capacitor-style native shell for `/drive` only (recommended), still Next.js + TypeScript | Client + Spike |
| **U-V1** | **What does "he could have many cars" mean?** (a) one driver who switches cars (e.g., sedan/SUV/black car) on different days or shifts, or (b) a fleet with additional drivers later? | (a) keeps the scheduler as is: the resource is still one driver, and the car is an attribute that sets the price. (b) is a different product (assignment/dispatch across drivers). It changes the data model now | Phase 4 schema | (a). Schema gets `vehicles`, `price_configs`, and per-time vehicle assignment. Multi-driver stays out of scope | Steve / Client |
| **U-V2** | **Does the customer ever choose the car?** | The layout system forbids a ride-tier selector. If cars have different prices and customers can pick, we need one. Otherwise the admin assigns the car and the customer just sees it | Phase 3 booking UI | No customer choice. Admin assigns the active car per day/time; price follows that car. Customer choice = a feature flag, off | Client |
| **U-L1** | **Approve the customer-link change?** Your format (last name + last four of phone) is guessable and exposes pickup address, trip, and live driver location. Proposed: `/pickup/johnson-4821-k9Xp2mQz` (your human-readable part kept, plus a random suffix). Also: is the link **per trip** or **one persistent link per customer**? | Security and privacy. About 10,000 possibilities per surname is trivially guessable, and two customers can share surname + last 4 | Phase 3 (checkout sends the link) | Per-trip link, random suffix, expires after the trip window, hashed at rest, plus a "find my ride" recovery by last name + last 4 + one-time code | Steve / Client |
| **U-C1** | **Client agreement:** does the client consent to his name, pricing, service area, and transcript being used in a public Skool course? Who owns the repo, the `blaqtaxxi.com` domain/DNS, the Vercel and Google Cloud projects, and the live Stripe account after handoff? | Publishing without consent is a real risk. Ownership decides where we deploy and whose accounts we create | Publishing any episode; Phase 0 deploy target; Phase 9 | Build on demo/staging only; record nothing that shows client data; do not publish until confirmed | Steve / Client |
| **U-B1** | **BLAQ brand assets and approvals:** logo/wordmark/favicon; the **Nissan Sentra photo** (you referenced `nissan-sentra.jpg`; it was not delivered to me); a driver photo; confirm the assumed canvas colors `#F5F6F7`/`#FFFFFF`; approve a darker text-gray (proposed `#686D71`, see BRAND.md); approve a warning/late-risk color (red is reserved for destructive actions) | The layout's gray text fails contrast (3.19:1 on white, 2.95:1 on canvas, AA needs 4.5:1), and there is no warning color for "running late" | Phase 0.5 (screens), Phase 5 | Use `#686D71` for text-gray in code behind a token, flagged for approval. Late-risk state = navy text + icon + label (no new color) | Client / Steve |

**Long-lead item to start now (not a blocker for the prompt, but on the critical path to launch):** **U-N1 (SMS).** The customer link is most naturally delivered by text. US SMS to real people needs carrier registration (A2P 10DLC) and consent language, which can take days to weeks. Verify with the provider and start early, or plan email-first at launch.

---

## 2. Needed before a specific phase

### Access, admin, and business

| ID | Question | Blocks | Default | Answerer |
|---|---|---|---|---|
| **U-A1** | Admin login method and strength: Google sign-in, email magic link, passkey; require 2FA? Allowed devices? | Phase 4 | Auth.js v5, single allow-listed identity, passkey or 2FA required before production | Client |
| **U-A2** | Will anyone besides the driver ever need admin (assistant, bookkeeper)? Roles? | Phase 4 | Two roles built in (`admin`, `driver`), one person holds both | Client |
| **U-C2** | Run costs and who pays: Vercel, Neon, Upstash, Google Maps Platform, Stripe fees, SMS, Sentry. A monthly budget ceiling and alerts | Phase 9 | Free/dev tiers through staging; budget alerts set before launch | Steve / Client |
| **U-C3** | Support and operations: who answers customer problems, who is paged when the site is down, hosting/maintenance agreement, after-handoff responsibilities | Phase 9 | Error alerts to the owner; runbook delivered at handoff | Steve / Client |
| **U-C4** | Data retention and deletion: how long contact info and trip logs are kept; how a customer requests deletion | Phase 9 | Contact fields deleted 90 days after the trip; trip totals kept for accounting | Client / counsel |
| **U-B2** | Legal business name, address, support phone/email, and anything a receipt must show (business name, tax details) | Phase 6 (receipts) | Placeholders behind admin config | Client |
| **U-B3** | Brand typeface, if any | Phase 0.5 | System font stack from the layout system | Client |
| **U-B4** | Voice/copy for customer messages, terms, footer | Phase 7 | Plain, direct, short. Editable in admin | Client |

### Vehicles, pricing, and scheduling

| ID | Question | Blocks | Default | Answerer |
|---|---|---|---|---|
| **U-V3** | Is a car's price configuration the same three-step ladder ($20 / $25 / $30 by trip time) with different amounts, or a different structure (per-mile, per-minute, minimum fare, airport fee)? | Phase 2 | Three-tier ladder per car, editable in admin; versioned | Client |
| **U-V4** | Car details to show customers: year, make, model, color, plate, seats, photo. The filename says **Sentra**; the transcript says black. Year and plate unknown | Phase 4/5 | Fields exist and are admin-editable; no plate in the repo | Client |
| **U-S1** | Confirm the cutoff reading (D-09): last *pickup* can be scheduled at his end time, ride may finish after | Phase 1 | As written in D-09 | Steve |
| **U-S2** | His real-world numbers: load and unload time, safety buffer, whether he waits at a pickup and for how long | Phase 1 | 3 min load, 2 min unload, 5 min buffer, 5 min wait (config) | Client |
| **U-S3** | Is "ride now" in scope for launch? (The transcript says DM him for a pickup right now.) | Phase 3 | Yes, if feasible | Client |
| **U-S4** | Airports (DFW, Love Field) in the service area? They have their own permits and curb rules | Phase 2 / 9 | Treated as out of scope until compliance is verified; shows "request a quote" | Client / counsel |
| **U-S5** | Party size, luggage, car seats, pets, wheelchair-accessible needs | Phase 3 | Max 4 passengers; no accommodations promised | Client |
| **U-S6** | Trips outside DFW: refuse, or "request a quote"? | Phase 3 | Refuse with a contact prompt | Client |
| **U-S7** | Round trips, wait-and-return, multiple stops, recurring weekly rides | later | Not in MVP | Client |
| **U-S8** | Cancellation dollar amounts and no-show wait (D-52, D-53): $5 / $10 / 5 min are my picks | Phase 6 | As written; one config file | Client |
| **U-S9** | Does the driver want a way to *decline* or move a booking that is already paid (emergency)? | Phase 4 | Driver cancel = 100% refund + customer notice | Client |

### Payments and notifications

| ID | Question | Blocks | Default | Answerer |
|---|---|---|---|---|
| **U-P1** | Who absorbs card fees on a $20 fare; tips or not; any promo codes | Phase 6 | Absorbed; no tips; no promos | Client |
| **U-P2** | The live Stripe account must belong to the client. Who creates it, and payout schedule/bank | Phase 9 | Test mode until done | Client |
| **U-P3** | Payment methods: card plus Apple Pay/Google Pay, or also Cash App/Zelle/cash | Phase 6 | Card + wallets via Stripe | Client |
| **U-N1** | SMS provider and start of carrier registration; is the link also emailed? | Phase 7 | Email + on-screen link at launch; SMS behind a seam | Client / Steve |
| **U-N2** | Which messages go out: confirmation (with link), T-30 reminder, late notice, no-show, receipt | Phase 7 | Those five | Client |

### Maps, driver app, and the customer screens

| ID | Question | Blocks | Default | Answerer |
|---|---|---|---|---|
| **U-M1** | Google Maps Platform: whose billing account, budget cap, which products (Maps JS, Routes, Places, Geocoding); current pricing is **not verified** | Phase 2 | Demo provider until a key exists; separate browser/server keys | Client / Steve |
| **U-M2** | What exactly the customer sees *during* the trip: car position, route line, both pins, ETA to drop-off, refresh rate | Phase 5 | All of those; 10 s poll | Steve / Client |
| **U-D2** | Driver's navigation app: Google Maps default; allow Waze/Apple Maps as a setting? | Phase 5 | Google Maps only; setting later | Client |
| **U-D3** | At pickup, how does the driver confirm the right person: ask the customer's last name, or a 4-digit code the customer shows | Phase 5 | Name check; PIN optional later | Client |
| **U-R1** | Keep the layout's 5-star rating (private to the owner)? | Phase 8 | Keep, optional, private | Client |
| **U-R2** | The layout has "Message driver". Replace with call/text? Whose number does the customer see (his personal number, or a masked one, which costs extra)? | Phase 5 | Tap-to-call/text with the number set in admin; masking is a flagged option | Client |
| **U-R3** | Receipt content and delivery: on the customer's page, also emailed, PDF? | Phase 6 | On page + emailed link | Client |

### Legal and compliance (launch gates, unverified)

| ID | Question | Blocks | Default | Answerer |
|---|---|---|---|---|
| **U-G1** | Regulatory status, insurance for paid rides, airport/venue permits, sales/other taxes, privacy/location-consent rules. **None verified.** Who verifies (attorney, insurer) and by when | Phase 9 (go-live) | Launch gate; no public launch until signed off in `COMPLIANCE_CHECKLIST.md` | Client / counsel |
| **U-G2** | Terms of service, privacy policy, and the location-sharing disclosure: who writes and approves them | Phase 9 | Drafts prepared for counsel review; not presented as legal advice | Client / counsel |

---

## 3. Spikes (settled by a test, not a decision)

| ID | Test | Settles | When |
|---|---|---|---|
| **SP-1** | Drive test on the **client's actual phone**: driver PWA sends pings while he navigates in Google Maps for 30 minutes; log the gap between pings, screen-locked and unlocked, on iOS/Android as applicable | U-D1: does the customer's live map keep moving | Phase 0.5 |
| **SP-2** | If SP-1 fails: proof of concept of a native shell with background location for `/drive` only | U-D1 (native shell scope and store distribution) | Phase 0.5 |
| **SP-3** | Compare Routes-API predicted travel times (with future departure) against 20 real drives at the driver's typical hours | Buffer and peak factors (U-S2); accuracy claims | Phase 5, then post-launch |
| **SP-4** | Google Maps deep link (`dir/?api=1&destination=…&dir_action=navigate`) opens and starts navigation on the driver's phone | Driver nav flow | Phase 0.5 |

## 4. Settled since the last list

| Item | Result |
|---|---|
| Build number | 032 (confirmed) |
| Lewisville → Dallas at $25 | Confirmed |
| Price tiers | Confirmed for MVP |
| Cutoff rule | Stated; my reading still to confirm (U-S1) |
| Map provider | **Google Maps** (stated). Google Maps URLs need no key (verified); Navigation SDK is native-only (verified) |
| Cancellation | Modeled on Uber/Lyft; only Lyft's 60-minute rule verified, amounts assumed |
| Vehicle model | **Sentra** (from the photo filename; the transcript's "Central" was a transcription error). Color black per transcript |
| Warning color | Red now exists in the palette but is reserved for destructive actions; a warning color is still open (U-B1) |

## 5. What I could not verify (say so, don't guess)

- The nissan-sentra.jpg file was not received.
- Google Maps Platform pricing and quotas; SMS carrier-registration lead times; Stripe's current fees.
- Whether Uber's help pages match my Lyft-based picks (Uber's pages returned 404).
- Any legal, insurance, permit, or privacy requirement.
- Background-location behavior on the client's phone (SP-1).
