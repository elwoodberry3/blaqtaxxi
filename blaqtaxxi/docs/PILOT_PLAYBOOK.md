# PILOT_PLAYBOOK.md — kick the tires, tune, approve

**Status:** written from the spec before the app exists. Phase 9 finalizes every checklist item against the built app and fills the driver's how-to guide with real screenshots (taken on `demo`/fake data, never with customer data).

## 1. What the pilot is

- The app running at **`blaqtaxxi.iasbootcamp.com`** on IAS's infrastructure (IAS owns the domain/DNS, Vercel, Google Cloud, and Stripe accounts during the pilot).
- The **client (the driver)** uses it with these instructions for **1–2 feedback rounds** to tune it to his exact needs.
- The **public may use it too, up to the moment money would be paid.** **No payment is taken and no ride is booked in the pilot.** Stripe is in test mode; only allow-listed testers can complete a test payment.
- When the client approves the final, IAS gives him instructions to deploy the app on **his own** domain and accounts (`docs/CLIENT_DEPLOY_GUIDE.md`).

## 2. Roles

| Role | Who | Can do |
|---|---|---|
| **Owner-driver (client)** | The driver | Admin, driver app, and customer flow, including test payments |
| **IAS (Steve)** | Builder | Deploys, tunes, triages feedback, records approval |
| **Allow-listed testers** | The client and anyone he names | Complete a Stripe **test** payment and receive test messages |
| **Public** | Anyone with the URL | Browse, choose a car, pick a time, enter details, reach the payment step, then stop |

## 3. What the public sees and what happens to their data

- A **pilot banner** on every page: "Pilot. No payment is taken and no ride is booked."
- At the payment step, a plain notice explaining that nothing is booked; the hold expires.
- Contact details entered before that point are **discarded when the unpaid hold expires (24 h at most)**. No SMS to the public; email only to allow-listed testers.
- A privacy notice and "pilot" terms (drafts for counsel; not legal advice, U-G1/U-G2).
- Public traffic can teach us which times and cars people try. Do **not** claim any usage numbers anywhere unless they are measured.

## 4. The rounds (time boxes are assumptions, U-F1)

| Step | Who | What | Time box |
|---|---|---|---|
| **Setup** | IAS | Deploy pilot, seed cars/hours/prices in the admin, upload photos, add testers to the allow-list, install the driver app on the driver's phone, send the client this playbook | before Round 1 |
| **Round 1** | Client | Work through the checklist in §5 and use it like a normal week, sending feedback from inside the app | about one week |
| **Triage 1** | IAS + client | 30-minute call: sort feedback (§7), agree what changes | after Round 1 |
| **Tune 1** | IAS | Config changes right away through the admin; behavior fixes shipped to the pilot; new ideas to the backlog | days |
| **Round 2** (optional) | Client | Re-test what changed plus anything new | about one week |
| **Approval** | Client + Steve | Written sign-off (§9) | after the last round |

## 5. Kick-the-tires checklist

Mark each item Works / Doesn't / Unsure and add a note. "Send feedback" is on every screen.

### A. Admin (`/admin`)
1. Sign in the way you were told (2FA or passkey). Sign out and in again.
2. Set your normal hours (default 5:00 am to midnight). Check that customers see them.
3. Change **one specific day** to end at 2:00 pm. Confirm no customer can pick a time after 2:00 pm that day, and that a ride starting at 1:50 pm is still allowed to run past 2:00 pm.
4. Try to shorten a day that already has a booking; confirm it warns you and does **not** cancel anything.
5. Add or edit a car: photo, model, seats, description. Mark one as an example if it is not yours.
6. Assign a car to a day. Try to overlap two cars; confirm it refuses. Use **Switch car** mid-day and confirm a 30-minute swap block appears.
7. Change a car's prices; read the preview; confirm **existing bookings keep their old price**.
8. Change the cancellation amounts and one message; confirm previews look right.
9. Export your configuration. (You will import it on your own deployment later.)
10. Open the feedback inbox and confirm your notes arrived.

### B. Driver app (on your phone)
1. Install it as instructed. Allow location "**Always**" (or the equivalent) and note every prompt you see.
2. Open today's schedule. Are the pickup times, gaps, and drive times what you expect?
3. Tap **Open in Google Maps** for the next pickup. Does Google Maps open and start directions?
4. Come back to the app, tap **Arrived**, then **Rider in car**. The app should now offer **Open in Google Maps** to the drop-off. Tap **Complete**.
5. With a test booking: what happens if the customer does not show (**No-show** after the wait)?
6. Is anything late-risk shown when you are running behind? Does the warning make sense?
7. Sign in on a second phone (if you have one). Confirm the first phone is signed out.
8. Drive with Google Maps in front for at least 20 minutes. Ask a tester to watch the customer page: does the car keep moving?

### C. Customer flow (`/book` and your link)
1. Start a booking: pickup, drop-off, date. Are the cars shown correctly, each with its own price?
2. Pick a car, then a time. Does anything look wrong or missing ("why is 9:00 gone?")?
3. Try a party too big for the small car; confirm it is not offered.
4. Pay with the Stripe **test** card you were given (allow-listed testers only).
5. Open your link. **Before:** are the trip details and countdown right? Does it tell you to check back 30 minutes before?
6. **30 minutes before:** does it show where the driver is? **During:** do you see the car on a Google Map and an estimated arrival? **After:** do you get a trip log and a receipt?
7. Cancel a test booking more than an hour before pickup, then one inside an hour; confirm the fee is shown before you confirm.
8. Try to guess someone else's link (change the random part). It must not work.
9. Lose your link and use "find my ride".

### D. Public trial (ask a friend who is not on the tester list)
1. They can browse, choose a car, pick a time, and enter details.
2. At the payment step they see the pilot notice and stop. No ride is booked.

## 6. How to send feedback

Use **Send feedback** on any screen. It attaches the screen, your role, and the environment. Write what you expected, what happened, and how much it matters:

| Type | Example |
|---|---|
| Bug | "Tapped Arrived and nothing changed" |
| Wrong behavior | "It offered 9:00 but I cannot make it from where the last ride ends" |
| Copy or look | "This wording is confusing" |
| Setting | "Buffer between rides should be 10 minutes" |
| New idea | "Let riders add a stop" |

Severity: **Blocks launch**, **Should fix**, **Nice to have**.

## 7. How feedback is triaged

| Kind | Handling |
|---|---|
| **Setting** (hours, prices, buffers, copy, policy amounts) | Changed at once through the admin; no code |
| **Behavior fix** (bug or wrong result) | Fixed this round if it blocks launch or affects safety/money; otherwise next round |
| **New feature** | Goes to the backlog; not part of this approval |
| **Blocks launch** | Must be resolved before approval |

## 8. What is deliberately not in the pilot

Real payments; SMS to the public; anything marked in the **Known limitations** page (generated from `docs/UNKNOWNS.md`); airport-specific handling; a customer account. These are listed so nobody is surprised.

## 9. Exit criteria and approval

The client approves the final when: every "Blocks launch" item is resolved; the checklist in §5 has no "Doesn't work" left that he has not accepted in writing; and the configuration in the pilot is what he wants to carry over.

```
BLAQTAXXI pilot approval
I have used the pilot and I approve the current version as final for launch,
with these accepted limitations: ______________________________
Name: ____________________   Date: ____________
Recorded by (IAS): ____________________   Date: ____________
```

Approval does **not** authorize live payments. Going live is a separate sign-off on `docs/LAUNCH_CHECKLIST.md` and `docs/COMPLIANCE_CHECKLIST.md` in his own deployment (Phase 10).

## 10. The driver's how-to guide (outline, finalized in Phase 9)

1. Installing the app (iPhone through TestFlight is the default; Android through internal testing), permissions to allow, battery settings to change.
2. Your day: schedule, next pickup, time to spare.
3. Going to a pickup: **Open in Google Maps**, then **Arrived**.
4. Picking up: confirm the customer's last name, **Rider in car**, then **Open in Google Maps** to the drop-off, then **Complete**.
5. When a customer does not show; when you are running late.
6. Switching cars and switching phones.
7. What customers see and when (30 minutes before pickup).
8. Who to contact.

## 11. Where each thing lives

Pilot URL and accounts: IAS. Feedback: in-app → n8n → shared sheet/Slack. Known limitations: `docs/UNKNOWNS.md`. Decisions: `docs/DECISIONS.md`.
