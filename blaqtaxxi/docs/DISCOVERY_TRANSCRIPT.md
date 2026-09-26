# Discovery Transcript (verbatim source of truth)

> Captured as given. Typos and speech-to-text errors are intentionally preserved. Interpretations live in the **Annotations** section at the bottom and in `DECISIONS.md`. Do not "clean up" the original.

---

BLAQTAXXI
One-to-One Uber

Discovery Transcript

[begin]

If you're in DFW, you need a ride, Dallas-Fort Worth, I'm doing $20. I don't care where you're at, I'm gonna come get you. Only way it's gonna be more than $20 is if I gotta take you somewhere that's farther than 30 minutes, that'll be $25 to $30.

Louisville, Lil' M, if you're trying to come to Dallas, I'll come get you, $25. If you want me to come pick you up right now, DM me, Black Nissan Central, I'm gonna come get you. Preferably, go ahead and hit me up an hour or two hours in advance.

You know, today Wednesday, you know you got an interview on Friday, go ahead, hit me up, reserve your spot, my website coming soon. If you're in DFW, you need a ride.

[end]

There is only one car, one driver and one fare at a time.
This is a one man uber.

People reserve their time and wait for him to arrive
The next person wants to know where he is and how far away he is
Someone with a schedule pick up tomorrow sees their pick up last 32 hrs aways or something like that.

The driver sees his schedule
Its Monday. ..pick up at 10am.. ..
The appointments have to account for where he is going to be coming from based on his schedule.
In a way he is "dropping his location" to the app and people can see this 30 minutes before they are scheduled for a pick up.

Payments. . .
Pretty straight forward.
You book it. .
You pay for it. .
It's a flat fee so that makes it simple.

If he am going from Lewisville to Downtown Dallas at 8am. . it's rush hour... we have to account for that.
Assume his next pick up is back in Lewisville.. .the next pick up has to give him time to get back on time.

It needs to be real time'ish
The next pick up could be downtown Dallas meaning he is right around the corner. . ..no need to wait an hour.

The driver sets his availability at 5am to Midnight everyday.

He needs to be able to fill his day up with rides.. as many as possible based on his location.

I want to demonstrate this using Claude Code in an advanced SKOOL course.
I want to direct it from the details above.

Create a the file system (CLAUDE.md, SKILLS.md, MCP, etc) that gives Claude everything it needs and answers as many questions up front as possible in order to being this to life.

---

## Annotations (interpretation, not source)

| # | Source phrase | Reading | Confidence |
|---|---|---|---|
| A1 | "Louisville, Lil' M, ... come to Dallas ... $25" | **Confirmed by Steve: Lewisville, TX.** Speech-to-text for **Lewisville** (a DFW-metro city; the later notes use Lewisville → Downtown Dallas). So: Lewisville → Dallas is quoted at **$25**, consistent with the ">30 min = $25–$30" tier. | High (confirm) |
| A2 | "Black Nissan Central" | Vehicle is a **black Nissan**. "Central" is likely a mis-transcription of a model name. Do **not** print a model name in the UI; say "black Nissan". | Medium |
| A3 | "DM me" / "my website coming soon" | Today bookings happen by DM. The product replaces DM back-and-forth with self-serve booking. | High |
| A4 | "hit me up an hour or two hours in advance" | Soft minimum lead time of 60–120 min for scheduled rides. Not a hard rule. "Ride now" is still offered. | Medium |
| A5 | "pick up last 32 hrs aways" | Rider countdown to a **future scheduled pickup** (e.g. "in 32 h"), not a distance. Distance/ETA appears only near pickup time. | High |
| A6 | "dropping his location" ... "30 minutes before" | Driver location is shared with a rider **only in the T-30 min window**. Countdown before that. | High |
| A7 | "Its Monday .. pick up at 10am" | Driver day view is a timeline of scheduled pickups. | High |
| A8 | "8am, Lewisville → Downtown Dallas, rush hour" | Travel time must be **time-of-day aware**. Canonical test scenario. | High |
| A9 | "next pick up back in Lewisville" | The **return leg** (deadhead) after a drop-off is part of feasibility. Canonical test scenario. | High |
| A10 | "next pick up could be downtown Dallas... right around the corner... no need to wait an hour" | Slots are **dynamic, not fixed blocks**. A drop-off near the next pickup makes a much earlier slot feasible. Canonical test scenario. | High |
| A11 | "5am to Midnight everyday" | Daily availability window. **Resolved (Steve):** the end time is the last time a ride can be *booked to start*; a ride may finish after it. Hours are editable per date (D-09, D-44). | High |
| A12 | "fill his day ... as many as possible based on his location" | Objective = maximize completed rides per day. MVP delivers this via dynamic slot feasibility; V2 adds nudges and gap-fill. | High |


---

## Client brief update, 2026-09-25 (verbatim from Steve)

> While this is episodic from a teaching POV, this is to be treated as a production grade build for a client. In this case it is "BLAQTAXXI". A single driver application.
>
> A configuration update: He has one car today (nissan-sentra.jpg) but in the future he could have many cars. Each car has a price configuration.
>
> Consider what each side sees:
>
> **The driver of BLAQTAXXI**
> 1. Admin: He logs into a secure backend that configures calendar, cars, prices and other relevant things that drive what the customer sees.
> 2. Now he is in the position of the Lyft driver. He sees the direction of where the pick up is (Google Maps is default). When he arrives and confirms he has the customer in his car he needs to see where to go.
>
> **The customer**
> When they pay they are sent a link (`www.blaqtaxxi.com/pickup/[CUSTOMER LAST NAME + LAST FOUR DIGITS OF THEIR PHONE NUMBER]`)
> Before: The details of the upcoming trip and to check back 30 minutes before the trip to see where the driver is and other relevant details
> During: Similar to the Lyft customer. They see themselves in route (Google Map) and an estimated arrival
> After: A log of the trip, completion receipt

Earlier answers (same day): Lewisville, TX; Build 32; cancellation/no-show should follow an average of Uber and Lyft; the driver's end time is the last time a ride can be booked to start, with per-day overrides (example: a Thursday ending at 14:00); price tiers correct for MVP; use plan mode for the first prompt.

| # | Interpretation | Confidence |
|---|---|---|
| A13 | "Nissan Sentra" (from the photo filename) confirms A2: the vehicle is a Sentra, and the transcript's "Central" was a transcription error | High |
| A14 | "Many cars, each with a price configuration": read as one driver, several cars (U-V1); price is per car | Medium |
| A15 | The link format is stated; a random suffix is proposed for security (U-L1) | High that it is guessable |


---

## Steve's answers, 2026-09-25 (evening; condensed from his message)

- **Driver's phone:** either; Android today, an iPhone soon. A thin native app shell is acceptable. "In older versions of Uber they would give you the option to open the route in Google Maps which would open the Google Maps app. The driver then uses this. The app from the drivers POV doesn't 'own' the map."
- **Many cars:** one driver who switches cars (sedan/SUV/black car) on different days or shifts.
- **Customer chooses the car:** yes, at booking, for "a more luxury ride that the single driver owns (nissan-sentra.jpg and the example SUV, chevrolet-suburban-3500hdheavy-duty.jpg)".
- **Customer link:** approved with the random suffix; **per trip**.
- **Delivery:** the MVP demo is deployed "on our side" (e.g. `blaqtaxxi.iasbootcamp.com`). The client is the driver and his clients are the passengers. IAS provides instructions on how to use it and how to "kick the tires" for a **1–2 round feedback loop**. He can use it, and give it to the public, "up until money is paid". IAS owns the domain/DNS, Vercel and Google Cloud projects, and the demo/live Stripe account. Once the client approves the final, IAS gives instructions to deploy on his own domain, Vercel, etc.
- **Brand assets supplied:** `wordmark__blaq.png`; wordmark typeface Momo Trust Display (Google Fonts); `favicon.jpg`; `nissan-sentra.jpg`; `profile.jpg` (driver photo); `chevrolet-suburban-3500hdheavy-duty.jpg`. **Confirmed** canvas `#F5F6F7`/`#FFFFFF`. **Approved** the darker text gray `#686D71`. **Approved** a warning/late-risk color.

| # | Interpretation | Confidence |
|---|---|---|
| A16 | "Demo/live Stripe account owned by IAS" is read as test mode in the pilot; whether IAS's live account ever takes the client's real fares is left open and defaulted to no (U-P2) | Medium |
| A17 | "They can use it up until money is paid" is read as: the public can go through the payment step's entry but no payment is taken and no ride is booked | Medium |
| A18 | The Suburban is "the example SUV" and is treated as an example car until confirmed (U-V5) | Medium |


---

## Steve's answers, 2026-09-25 (late evening)

- **IAS's live Stripe account:** "No, IAS is always in Test mode. The objective is to show the client 'it works'."
- **The Suburban:** "not real. It is an example." It is "a placeholder to show the client they can manage their 'fleet' in the admin area. They can set prices for each vehicle." For its placeholder prices, "use a standard flat rate for riding 'Black' in apps like Uber or Lyft."
- **Switching cars mid-day:** "No. Cars are assigned by day or shift, and a change creates the swap block."
- **Store accounts:** "IAS owns the Apple and Google developer account in a test flight state. IAS never publishes the app."
- **iPhone:** "The iPhone is not real. We are planning ahead as if it is." "iPhone is the default though TestFlight internal testing."

| # | Interpretation | Confidence |
|---|---|---|
| A19 | Uber and Lyft do not publish a flat Black rate, so "standard flat rate for Black" is read as a clearly-labeled placeholder ladder, anchored to third-party Dallas estimates ($65 / $85 / $110) | Medium |
| A20 | "Google developer account in a test flight state" is read as the Play internal testing track (unverified) | Medium |
| A21 | "Planning as if it is real" means iOS is designed, built, and documented as the default now, even though no physical iPhone is yet involved. Whether a physical iPhone exists for the drive test is still open (U-D7) | Medium |
