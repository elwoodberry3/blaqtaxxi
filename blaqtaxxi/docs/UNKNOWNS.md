# UNKNOWNS.md — what is still open, and what it blocks

Updated 2026-09-25 (late evening), after Steve's answers on IAS's Stripe, the example SUV, car switching, store accounts, and the iPhone.

**How to read this.** Section 0 is what your answers settled. Section 1 is the short list that still changes what Claude *plans*. Section 2 is needed before a specific phase. Everything else has a safe default that Claude builds on and logs.

**Rule for Claude:** never assume an item marked **BLOCKING**. Stop and ask (`AskUserQuestion`). Defaulted items are built on the default and logged in `DECISIONS.md` as `assumed`.

Legend: **Answerer** = who can settle it. *Steve* = you. *Client* = the owner-driver. *Spike* = settled by a test.

---

## 0. Settled

| Item | Result |
|---|---|
| **U-D1** Driver's phone | **Either.** Android today, an iPhone soon. **A thin native shell is acceptable.** The driver app does not own the map; it opens Google Maps. So the shell is required to keep sending location while Google Maps is in front. Both platforms from day one |
| **U-V1** Many cars | One driver who switches cars by day or shift |
| **U-V2** Customer chooses the car | **Yes**, at booking, from the cars assigned for that time. Cars: black Nissan Sentra (today) and a luxury SUV shown as an example |
| **U-L1** Customer link | **Approved:** `/pickup/johnson-4821-k9Xp2mQz`, readable prefix plus random suffix, **per trip** |
| Delivery model | Pilot on IAS accounts at `blaqtaxxi.iasbootcamp.com`; 1–2 client feedback rounds; public may use it up to the payment step; the client then deploys on his own accounts from IAS's instructions |
| Ownership during the pilot (part of **U-C1**) | IAS owns the domain/DNS, Vercel, Google Cloud, and Stripe accounts |
| **U-B1** Canvas colors | **Confirmed** `#F5F6F7` / `#FFFFFF` |
| **U-B1** Text gray | **Approved** `#686D71` (`blaq-gray-text`) |
| **U-B1** Warning color | **Approved in principle.** No hex was given; I propose `blaq-amber #A15C00` (5.19:1 on white, 4.80:1 on canvas). One-line confirmation wanted |
| **U-B3** Brand typeface | **Momo Trust Display** (Google Fonts). Confirmed to exist as a Google Font; weights and license are **not verified** (fetch blocked) |
| Brand assets (U-B1) | Received: wordmark, favicon, Sentra, Suburban, driver photo (filed in `client-assets/`; findings in Section 3) |
| **U-P2** IAS Stripe | **IAS is always in test mode.** The objective is to show the client "it works." IAS never takes real fares; the client uses his own live Stripe account in his own deployment |
| **U-V5** The cars | **The Suburban is not real; it is an example** that shows the client he can manage a fleet in the admin and set a price per vehicle. Placeholder ladder **$65 / $85 / $110**, anchored to third-party Uber Black estimates for Dallas (Uber and Lyft publish no flat Black rate; the estimates are unverified). Editable in admin |
| **U-V6** Switching cars | **No switching within a shift.** Cars are assigned by day or shift, and a change creates the swap block |
| **U-D4** Store accounts | IAS owns the Apple Developer and Google developer accounts **in a testing state** (iOS: TestFlight internal testing; Android: Play internal testing, assumed). **IAS never publishes the app.** The client's own store accounts and any publishing are his decision after handoff |
| **U-D8** Which phone is the default | **iPhone**, through TestFlight internal testing. The iPhone is not real yet; we plan as if it is. Android stays supported (his phone today) |
| Earlier | Build 032 · Lewisville, TX · price tiers · Google Maps · vehicle is a Sentra |

---

## 1. Answer before firing the first prompt

Two things are left, and neither stops the plan-mode prompt. Both change what Phase 0.5 can prove.

| ID | Question | Why it matters | Default if unanswered | Answerer |
|---|---|---|---|---|
| **U-D7** | **Does IAS have a physical iPhone and a Mac (with Xcode) for the iOS drive test and the TestFlight builds?** The iPhone is not real for the client, but the spike and the TestFlight build still need a real device and a Mac (or a macOS build service, unverified) | iOS is the default platform, and the whole point of the shell is that location keeps flowing while Google Maps is in front. My understanding is that a simulator cannot show real background behavior on a real drive (unverified), so without a physical iPhone the iOS path would ship **built but unproven** | Run SP-1 on Android now (the client's real phone), build the iOS shell, and label iOS **unverified** everywhere until a physical iPhone is used | Steve |
| **U-V5b** | **Confirm the placeholder SUV prices: $65 / $85 / $110.** I could not get a published Black rate, so I anchored to third-party Dallas estimates (a $7 base, $3.51 per mile, $0.35 per minute; example flat trips of $75–$120). A $65 minimum for the SUV next to a $20 Sentra will look steep; that is fine for a placeholder, but say if you want different numbers | What the client sees in the fleet demo | As written | Steve |

---

## 2. Needed before a specific phase

### Pilot and handoff

| ID | Question | Blocks | Default | Answerer |
|---|---|---|---|---|
| **U-F1** | Feedback loop mechanics: how does the client submit feedback (in-app "Send feedback" → n8n → a shared sheet), how long is each of the 1–2 rounds, what is in scope (config, behavior, new features), who triages, and what does "client approves the final" mean in writing | Phase 9 | `docs/PILOT_PLAYBOOK.md` as written: in-app feedback, one-week rounds, config/behavior in scope, new features to a backlog, written sign-off | Steve / Client |
| **U-H1** | Handoff: can the client (or someone he uses) follow a deploy guide, or should IAS assist live? Who owns the code and IP after handoff (licence terms)? Ongoing support and maintenance? | Phase 10 | Guide + one assisted walkthrough; ownership per a written agreement (open) | Steve / Client |
| **U-C1** | Course publication: does the client consent to his brand, pricing, photos, and transcript appearing in a public Skool course? | Publishing any episode | Record on `demo` with fake data only; publish nothing with his real data | Steve / Client |
| **U-C2** | Run costs during the pilot (Vercel, Neon, Upstash, Google Cloud, SMS, storage) and a monthly ceiling; who pays at handoff | Phase 9 | Free/dev tiers, budget alerts | Steve |
| **U-C3** | Support during the pilot: who answers a member of the public who reaches the payment step and expects a ride | Phase 9 | Pilot banner + "no rides are booked in the pilot" copy | Steve / Client |
| **U-C4** | Data retention/deletion, including public data entered in the pilot | Phase 9 | Unpaid-hold contact data purged within 24 h; paid test bookings deleted at pilot end | Steve |

### Admin and access

| ID | Question | Blocks | Default | Answerer |
|---|---|---|---|---|
| **U-A1** | Admin login method and strength (Google sign-in, magic link, passkey); require 2FA | Phase 4 | Auth.js v5, one allow-listed identity, passkey or 2FA before the public sees the pilot | Client |
| **U-A2** | Anyone besides the driver needing admin access | Phase 4 | Two roles in code, one person holds both | Client |
| **U-M1** | Google Cloud (IAS-owned for the pilot): budget cap, products enabled (Maps JS, Routes, Places, Geocoding). Pricing **not verified** | Phase 2 | Demo provider until keys exist; separate browser/server keys | Steve |

### Cars, scheduling, pricing

| ID | Question | Blocks | Default | Answerer |
|---|---|---|---|---|
| **U-V3** | Is each car's price the same three-step ladder with different amounts, or a different structure (per-mile, minimum, airport fee) | Phase 2 | Three-tier ladder per car | Client |
| **U-V4** | Car details to show: year, make, model, color, plate, seats, photo | Phase 4 | Admin fields; no plate in the repo | Client |
| **U-S1** | Confirm the cutoff reading (D-09): last *pickup* at his end time, ride may finish after | Phase 1 | As written | Steve |
| **U-S2** | His real load/unload time, buffer, and wait | Phase 1 | 3 / 2 / 5 / 5 min (config) | Client |
| **U-S3** | Is "ride now" in scope | Phase 3 | Yes, if feasible | Client |
| **U-S4** | Airports (DFW, Love Field) in the area | Phase 2 / 10 | Out of scope until compliance is verified | Client / counsel |
| **U-S5** | Party size, luggage, car seats, accessibility | Phase 3 | Party size limited by the chosen car's seats; no other accommodations promised | Client |
| **U-S6** | Trips outside DFW: refuse, or "request a quote" | Phase 3 | Refuse with a contact prompt | Client |
| **U-S7** | Round trips, multiple stops, recurring rides | later | Not in MVP | Client |
| **U-S8** | Cancellation dollar amounts (D-52, D-53): $5 / $10 / 5 min are my picks | Phase 6 | As written | Client |
| **U-S9** | Driver cancelling or moving a paid booking | Phase 4 | Driver cancel = 100% refund + customer notice | Client |

### Payments, notifications, media

| ID | Question | Blocks | Default | Answerer |
|---|---|---|---|---|
| **U-P1** | Who absorbs card fees on a $20 fare; tips; promo codes | Phase 6 | Absorbed; no tips; none | Client |
| **U-P3** | Payment methods at production: card + Apple/Google Pay, or also Cash App/Zelle | Phase 6 | Card + wallets | Client |
| **U-N1** | SMS provider and carrier registration (lead time **not verified**). **In the pilot no SMS goes to the public** | Phase 7 | Pilot: on-screen link for test bookings, email only to allow-listed testers. SMS behind a seam | Client / Steve |
| **U-N2** | Which messages go out | Phase 7 | Confirmation (with link), T-30 reminder, late notice, no-show, receipt | Client |
| **U-B2** | Legal business name, address, support contact, receipt details | Phase 6 | Admin placeholders | Client |
| **U-B4** | **Wordmark assets:** a transparent-background or SVG wordmark, and a reversed (white) version for navy headers and the driver card. The supplied PNG is 195×75, opaque, black on white | Phase 0.5 | Place the supplied PNG on white only; flag as a visible limitation | Client / Steve |
| **U-B5** | **Real photos of the cars.** The Sentra and Suburban images look like manufacturer stock images (not verified); use the driver's real cars for a real launch | Phase 4 | Use the supplied images in the pilot only if the client confirms he is comfortable; label the Suburban "example" | Client |
| **U-B6** | Voice and copy for customer messages, terms, footer | Phase 7 | Plain, direct, short; editable in admin | Client |
| **U-B7** | **A better driver headshot.** `profile.jpg` is a tilted, fisheye-distorted selfie. Riders identify drivers by photo | Phase 5 | Crop to the face for the pilot; recommend a straight-on, well-lit photo | Client |
| **U-B8** | **Favicon source.** `favicon.jpg` is a 512×512 JPEG, soft, no transparency | Phase 0 | Generate sizes from it for the pilot; request an SVG/PNG master | Client |
| **U-B9** | Body typeface: Momo Trust Display for the wordmark and headings; is there a text face for UI, or the system stack | Phase 0.5 | System stack for UI text | Client |

### Driver app, maps, customer screens

| ID | Question | Blocks | Default | Answerer |
|---|---|---|---|---|
| **U-D6** | Shell architecture: load the hosted `/drive` in the shell (fast to build, instant updates) or bundle a static driver UI. **IAS never publishes the app, so store-review risk for a wrapped web page only matters if the client later publishes it himself** | Phase 5 | Hosted `/drive` | Steve |
| **U-M2** | What the customer sees during the trip | Phase 5 | Car on the route, both pins, ETA to drop-off, 10 s refresh | Steve / Client |
| **U-D2** | Navigation app: Google Maps only, or Waze/Apple Maps as a setting | Phase 5 | Google Maps only | Client |
| **U-D3** | Confirming the customer at pickup | Phase 5 | Driver asks the last name; a code is a later option | Client |
| **U-R1** | Keep the optional, private 5-star rating | Phase 8 | Keep | Client |
| **U-R2** | "Message driver": tap-to-call/text on which number (his personal number, or masked, which costs extra) | Phase 5 | Number set in admin; masking later | Client |
| **U-R3** | Receipt content and delivery | Phase 6 | On page + emailed link | Client |

### Legal (launch gate for the client's own deployment; unverified)

| ID | Question | Blocks | Default | Answerer |
|---|---|---|---|---|
| **U-G1** | Regulatory status, insurance for paid rides, permits, taxes, privacy/location-consent. **None verified.** Also: the pilot collects real public data before payment, and shows real availability and pricing; what notice and terms does that need | Phase 9 (pilot to the public) and Phase 10 | Pilot privacy notice + "pilot, no rides booked" terms drafted for counsel; go-live gated on `COMPLIANCE_CHECKLIST.md` | Client / counsel |
| **U-G2** | Terms of service, privacy policy, and location-sharing disclosure: who writes and approves | Phase 9 / 10 | Drafts for counsel review; not presented as legal advice | Client / counsel |

---

## 3. Findings from the client assets (filed in `client-assets/`)

| File | Finding | Action |
|---|---|---|
| `wordmark__blaq.png` | 195×75 PNG, **fully opaque** (a white background is baked in), black lettering. Black on the navy header is 1.31:1, so it cannot go on navy. Soft on high-density screens | Request SVG/transparent + reversed version (U-B4) |
| `favicon.jpg` | 512×512 JPEG, black "B", soft, no transparency | Generate 16/32/180/192/512 for the pilot; request a master (U-B8) |
| `nissan-sentra.jpg` | 1080×1080, black Nissan sedan on white. An "SR" badge is visible on the grille; trim not confirmed | Use as the Sentra image; confirm model/year (U-V5) |
| `chevrolet-suburban-3500hdheavy-duty.jpg` | 1080×1080, dark Chevrolet Suburban on white, "High Country" badge visible; filename says 3500HD | Treated as an **example** car (U-V5) |
| `profile.jpg` | 1080×1080, tilted, fisheye-distorted selfie | Crop for the pilot; request a headshot (U-B7) |
| All | Kept out of git (`client-assets/` is gitignored); uploaded through the admin | Media storage decided in `DECISIONS.md` |

## 4. Spikes (settled by a test, not a decision)

| ID | Test | Settles | When |
|---|---|---|---|
| **SP-1** | On a **real iPhone (the default) and a real Android phone**: the native shell sends location every 15 s while the driver navigates in Google Maps for 30 minutes, screen unlocked and locked. Log ping gaps, battery drain, and OS prompts/notifications. Simulators are not evidence (my understanding, unverified). **Needs a physical iPhone and a Mac (U-D7)** | Whether the shell keeps the customer's map moving (U-D1 follow-through) | Phase 0.5 |
| **SP-2** | Distribution dry run: iPhone through **TestFlight internal testing** on IAS's Apple Developer account, Android through the Play internal testing track (assumed) or a direct install. Record every step, cost, and blocker (tester caps, build expiry, whether the driver must be an App Store Connect user are unverified) | U-D4 follow-through | Phase 0.5 |
| **SP-3** | Compare routing-API predicted times (future departure) with 20 real drives at his usual hours | Buffer and peak factors (U-S2) | Phase 5, then during the pilot |
| **SP-4** | The Google Maps deep link opens the Maps app and starts navigation on both phones | Driver flow | Phase 0.5 |

## 5. What I could not verify (say so, don't guess)

- The exact weights and license of **Momo Trust Display** (the page confirmed the family is on Google Fonts; details did not load, GitHub was blocked).
- Apple/Google developer program rules and costs, TestFlight internal-testing limits (tester caps, build expiry, whether the driver must be an App Store Connect user), the Play internal-testing equivalent, and any background-location plugin's behavior.
- What Uber and Lyft charge for Black: neither publishes a flat Black rate. The SUV placeholder is anchored to third-party estimates (RideWise, TaxiFareFinder) that I could not verify.
- Whether a simulator can stand in for a real iPhone on a real drive (my understanding is no).
- Google Cloud/Maps pricing, Vercel and storage limits, SMS registration lead times, Stripe's current fees.
- Whether the two car images are stock images (they look like it).
- Every legal, insurance, permit, and privacy requirement.
- That Uber's cancellation policy matches my Lyft-based picks (Uber's pages returned 404).
