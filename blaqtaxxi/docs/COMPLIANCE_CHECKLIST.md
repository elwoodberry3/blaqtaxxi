# COMPLIANCE_CHECKLIST.md — verify before any real launch

**Status: UNVERIFIED. This is a list of questions, not answers, and not legal advice.** I don't know the current state of these rules and have not confirmed any of them. Everything below needs to be checked with a licensed attorney, the insurer, and the relevant agencies. For the **course demo** (fake data, test-mode payments, no real riders) none of this blocks the build; for a **real launch** all of it does.

Claude Code must not describe any item here as settled, and must not claim the product is "compliant" or "licensed."

## 1. Is this a regulated ride service?

- [ ] Does a single driver taking paid rides booked through an app count as a transportation network company (TNC) or similar under Texas law? Which agency regulates it, and what does registration require? (My understanding is that Texas regulates TNCs at the state level, but that is **unverified**.)
- [ ] Does booking by app change his status compared with taking DMs today, or is the legal question the same either way (paid rides for hire)?
- [ ] Are there city-level requirements (Dallas, Fort Worth, Lewisville, others) that apply regardless?
- [ ] Driver requirements: license class, background checks, vehicle inspection, age of vehicle, permits.

## 2. Insurance

- [ ] Does his personal auto policy cover paid passenger rides? (Many personal policies exclude commercial use; **verify with the insurer**, in writing.)
- [ ] What coverage does a for-hire or TNC-style policy require, and what does it cost?
- [ ] Coverage during the "deadhead" leg (driving to a pickup with no rider) vs with a rider.

## 3. Airport and venue access

- [ ] DFW Airport and Love Field pickup/drop-off rules, permits, and fees. The scheduler does not model airport queues or curb rules; the PRD marks airport logic out of scope.
- [ ] Stadiums, event venues, hospitals: any restrictions.

## 4. Payments

- [ ] Confirm current Stripe fees and how a **flat $20 fare** is affected; decide who absorbs the fee. (Fees are unverified here; check the live pricing page.)
- [ ] Refund and dispute handling; chargeback policy; receipts.
- [ ] Sales tax or other taxes on the service; business entity and licensing.
- [ ] Never store card data ourselves: Stripe Elements/Checkout only.

## 5. Privacy and location data

- [ ] Precise geolocation of the driver and riders is likely **sensitive personal data** under some state privacy laws. Check Texas requirements (consent, notice, retention) before collecting it from real people.
- [ ] Privacy policy and terms of service, including the T-30 location-sharing disclosure, ping retention (24 h max in this design), and contact-data deletion (default 90 days, see ARCHITECTURE.md §3).
- [ ] Data access/deletion request process.

## 6. Messaging

- [ ] US SMS needs carrier registration (A2P 10DLC) and consent handling; check lead time and TCPA-style consent language before texting real people.
- [ ] Email: sender verification, unsubscribe rules (transactional vs marketing).

## 7. Accessibility and safety

- [ ] ADA and service-animal rules for a for-hire vehicle; WCAG AA on the web app.
- [ ] Rider safety features (share-my-trip, emergency contact) — not in MVP; decide whether a real launch needs them.
- [ ] Incident and complaint process.

## 8. Claims and marketing copy

- [ ] Do not claim "licensed," "insured," "safe," or "verified" anywhere in the UI unless it is true and documented.
- [ ] No fabricated reviews, ratings, ride counts, or testimonials (project brand rule).

## "Policy to Code" episode tie-in

Separate **business policy** (cancellation windows, grace period, buffers: config files and tests, owned by the operator) from **legal requirements** (permits, insurance, data rules: owned by counsel, encoded only after verification). The episode shows the first as `policy.config.ts` + tests, and the second as this checklist, honestly marked unverified.

## Sign-off (for a real launch)

| Item | Owner | Verified on | Evidence link |
|---|---|---|---|
| Regulatory status |  |  |  |
| Insurance |  |  |  |
| Airport/venue permits |  |  |  |
| Payments & tax |  |  |  |
| Privacy & location consent |  |  |  |
| SMS registration |  |  |  |
