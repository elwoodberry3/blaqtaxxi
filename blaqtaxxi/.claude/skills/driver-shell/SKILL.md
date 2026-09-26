---
name: driver-shell
description: Use when building or changing native/driver-shell, the thin native app (Android and iPhone) that loads /drive, keeps sending location in the background while Google Maps is in front, handles device registration, and is distributed to the driver. Includes the spike protocol.
---

# driver-shell

Read `docs/ARCHITECTURE.md` §8, `docs/DECISIONS.md` D-40, D-41, D-109, D-110, and `docs/UNKNOWNS.md` (SP-1, SP-2, SP-4, U-D4, U-D6) first.

## Why it exists

The driver app does **not own the map**: it opens the Google Maps app (like older Uber versions). While Google Maps is in front, a plain web page is backgrounded and stops reporting location, which would freeze the customer's live map. The shell keeps sending location in the background. *Verified:* Google's Navigation SDK is Android/iOS (plus Flutter and React Native) only, with no web version; Google Maps URLs start navigation with no API key.

**Approved by Steve:** a thin native shell is acceptable. **iPhone is the default platform**, delivered by TestFlight internal testing (the iPhone is not real yet, so plan as if it is), and **Android is supported** (his phone today). **IAS never publishes the app:** IAS owns the Apple Developer and Google developer accounts in a testing state (TestFlight internal testing; the Play internal testing track, assumed). What the client does with his own store accounts after handoff is his decision.

## What is NOT verified (say so, do not guess)

- Which background-location plugin works best; evaluate at least two candidates in the spike and report what you could and could not confirm. Do not assume a plugin's behavior from its README.
- Android foreground-service and battery-optimization behavior on the driver's actual phone.
- iOS background-location permission wording and the review treatment of a shell that loads a hosted page (App Store guideline on minimum functionality), which only matters if the client later publishes it himself; TestFlight internal-testing rules, build expiry, and costs.
- Google Play vs sideloaded distribution rules and costs.
- Whether a hosted-URL shell or a bundled static UI is better (U-D6). Default for the pilot: **hosted `/drive`**.
- Whether a simulator can stand in for a real iPhone: my understanding is that background execution and suspension behave differently on a real device on a real drive, so treat simulator runs as **non-evidence** for SP-1 (unverified). A physical iPhone and a Mac are needed (U-D7); if none is available, iOS stays labeled unverified.
- TestFlight internal-testing limits (tester caps, build expiry, whether the driver must be an App Store Connect user) and the Play internal-testing equivalent.

## Scope of the shell (keep it thin)

1. Loads the hosted `/drive` (allow-list navigation to our origin only; nothing else gets the native bridge).
2. **Background location** with a persistent, honest notification on Android; correct permission strings on iOS. Posts to `/api/driver/ping` with `deviceId` on a 15 s cadence (config), and on every status tap.
3. **Device registration** (`POST /api/driver/device`): first launch registers; a new device requires re-auth and revokes the old one (D-110).
4. Opens Google Maps via the deep link from `lib/navigation`.
5. Visible states for: location permission missing/"only while using", battery optimization on, offline, and "another device is active".
6. **No business logic, no secrets, no customer data stored in the shell.** The session token lives in secure storage. Do not add features to the shell that belong in `/drive`.

## Constraints

- **Do not set `output: 'export'`** in the Next.js app (gotcha 1). The shell loads the hosted app; if U-D6 chooses a bundled UI, that is a **separate static package**, not the main app.
- **Security:** lock the WebView to our origin; disable arbitrary navigation; HTTPS only; no debug builds in distribution.
- **iOS builds need a Mac, Xcode, and an Apple developer account. Claude in a cloud session cannot build or install them.** Prepare the project, the exact commands, and a checklist; Steve runs them. Same for Android Studio/emulator/device installs unless the CLI build works in the environment (say what you verified).
- Keep dependencies minimal and pinned. Record the versions used in the spike report.

## Spike protocol (SP-1, SP-2, SP-4)

1. Build the shell with a `/spike` page: request background location, ping every 15 s with `deviceId` and a timestamp, show ping-gap stats (min, median, max) and counts per 5-minute window, and a button that opens the Google Maps deep link to a fixed destination.
2. On **each** platform: install on a real phone; drive for 30 minutes with Google Maps in front, screen unlocked; repeat with the screen locked. Record gaps, battery drop, and every OS prompt or notification.
3. **SP-2:** install through the intended route (Android sideload/internal testing; iPhone via TestFlight internal testing), note every step, cost, and blocker.
4. **SP-4:** confirm the deep link opens Google Maps and starts navigation.
5. Write results into `docs/UNKNOWNS.md` (SP-1, SP-2, SP-4) and settle U-D6. **Do not claim a result you did not measure.**

## Definition of done

Spike results recorded from real phones · shell builds reproducibly with documented commands · revoked-device pings rejected server-side · permission/battery/offline states visible · no secrets or client data in the shell · distribution route documented for the pilot and for the client's own accounts (U-D4).
