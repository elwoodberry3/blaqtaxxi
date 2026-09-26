# SCHEDULER_SPEC.md — the heart of BLAQTAXXI

Everything else in this build is plumbing around this module. Build it first, test-first, and keep it pure.

## 1. Contract

`lib/scheduler/**` is a **pure domain module**:

- No imports from `next/*`, `drizzle-orm`, `stripe`, `fetch`, `process.env`, or any I/O.
- Time comes in as arguments (`now`), never `new Date()` inside logic.
- Travel time comes in through an injected `TravelTimeProvider`. The provider is `async` (real routing is a network call), so scheduler functions are `async`, but they do nothing except `await provider.minutes(...)` and do math.
- Tests use a table-driven fake provider (`docs/fixtures/travel-matrix.json`). Production uses `lib/routing`.

This is what makes it testable, explainable on camera, and reusable for any solo operator who travels between appointments.

## 2. Vocabulary

| Term | Meaning |
|---|---|
| **Item** | Something on the driver's day: a `ride` or a `block` (lunch, car wash) |
| **Car** | One of the driver's vehicles, with its own price configuration. The **customer chooses** it at booking (D-82); it is offered only inside its assignment windows (D-84). It changes price and display, not travel-time feasibility (D-83) |
| **Swap block** | An ordinary block at the car base (`home_place`) when the driver changes cars mid-day (default 30 min). The engine already handles blocks (S8), so it accounts for the trip to the car |
| **Start place / end place** | Where the driver must be when the item starts / where he is when it ends. Ride: pickup → dropoff. Block: same place both |
| **Dwell** | Load time at pickup (3 min) and unload time at drop-off (2 min) |
| **Deadhead** | Driving with no rider: from previous item's end place to this item's start place |
| **Buffer** | Safety margin added to every deadhead (5 min) |
| **Slack** | Minutes to spare after buffer. `< 0` means infeasible |
| **Gap** | The open interval between two consecutive items (or day edge) where a new ride could be inserted |
| **T-30 window** | 30 min before `pickupAt` until ride end: the only time a rider may see live location |

## 3. Config (all in `lib/config/scheduler.ts`, all overridable in tests)

```ts
export interface SchedulerConfig {
  tz: 'America/Chicago';
  defaultDayStartMin: 300;       // 05:00 (per-date overrides beat this, D-44)
  defaultDayEndMin: 1440;        // 24:00 = last time a ride may be BOOKED TO START (D-09)
  // Cutoff rule: pickupAt <= day end (inclusive). A ride may finish after the cutoff.
  bufferMin: 5;                  // D-21
  loadDwellMin: 3;
  unloadDwellMin: 2;
  slotStepMin: 5;                // D-22
  minLeadMin: 60;                // D-10
  maxHorizonDays: 14;
  visibilityWindowMin: 30;       // D-31
  maxProviderCallsPerSearch: 200;
  peak: [ ['06:30','09:30'], ['15:30','19:00'] ]; // used by the DEMO provider only
}
```

## 4. Types (starting point; refine as you build)

```ts
export interface LatLng { lat: number; lng: number }
export interface Place { id: string; label: string; point: LatLng }

export interface TravelTimeProvider {
  /** Whole minutes, rounded up. 0 if from and to are the same place. */
  minutes(from: Place, to: Place, departAt: Date): Promise<number>;
}

export type Item =
  | { kind: 'ride'; id: string; pickup: Place; dropoff: Place;
      pickupAt: Date; plannedEndAt: Date }        // stored at booking; never recomputed
  | { kind: 'block'; id: string; at: Place; startAt: Date; endAt: Date };

export interface DayContext {
  now: Date;
  homeBase: Place;                                 // D-12
  window: { start: Date; end: Date };              // from dayWindow(date, tz)
  driverNow?: { place: Place; at: Date };          // live position, for ride-now and late-risk
  vehicleWindows: Record<string, { from: Date; to: Date }[]>;  // per-car assignment windows (D-84); no overlaps
}

export type Reason =
  | 'OUTSIDE_AVAILABILITY'
  | 'TOO_SOON'
  | 'OVERLAP'
  | 'CANT_REACH_PICKUP'      // previous item + deadhead + buffer > pickupAt
  | 'BREAKS_NEXT_PICKUP'     // this ride's end + deadhead + buffer > next start
  | 'CAR_NOT_ASSIGNED';      // pickup is outside the chosen car's assignment windows (D-112)

export type InsertResult =
  | { ok: true;  plannedEndAt: Date; rideMinutes: number;
      deadheadInMin: number; slackInMin: number;
      deadheadOutMin: number; slackOutMin: number; tightFit: boolean }
  | { ok: false; reason: Reason; shortfallMin?: number };
```

## 5. Feasibility: `canInsert(candidate, schedule, ctx, provider, cfg)`

The candidate is `{ vehicleId, pickup, dropoff, pickupAt, rideNow? }`. The existing `schedule` is assumed **already valid** and sorted by start time.

1. **Availability.** The `window` is that date's effective day: the default 05:00–24:00, or the driver's override for that date (shortened, moved, or closed; D-44). Require `window.start <= pickupAt <= window.end`. **The end time is a cutoff on when a ride may *start*** (D-09); `plannedEndAt` may fall after it. Compute `rideMinutes = provider.minutes(pickup, dropoff, pickupAt + loadDwell)` and `plannedEndAt = pickupAt + loadDwell + rideMinutes + unloadDwell` (needed for steps 3 and 5). A closed day has no window: every candidate fails. Fail → `OUTSIDE_AVAILABILITY`.
1b. **Car.** Require `pickupAt` to lie inside one of `ctx.vehicleWindows[vehicleId]`. Fail → `CAR_NOT_ASSIGNED`. (Everything below is car-independent: only the driver's travel matters.)
2. **Lead time.** Unless `rideNow`, `pickupAt >= now + minLeadMin`. Fail → `TOO_SOON`.
3. **Overlap.** No existing item's `[start, end)` intersects `[pickupAt, plannedEndAt)`. Fail → `OVERLAP` (takes precedence over 4 and 5).
4. **Reach the pickup (predecessor).** `prev` = the item with the latest `end <= pickupAt`.
   - Origin = `prev.endPlace` and `departAt = prev.endAt`.
   - No `prev`: origin = `driverNow?.place ?? homeBase`, `departAt = max(window.start, driverNow?.at ?? window.start)`.
   - `deadheadIn = provider.minutes(origin, pickup, departAt)`.
   - Require `departAt + deadheadIn + bufferMin <= pickupAt`. Else `CANT_REACH_PICKUP`, `shortfallMin = arrival + buffer - pickupAt`.
5. **Make the next pickup (successor).** `next` = the item with the earliest `start >= plannedEndAt`.
   - `deadheadOut = provider.minutes(dropoff, next.startPlace, plannedEndAt)`.
   - Require `plannedEndAt + deadheadOut + bufferMin <= next.startAt`. Else `BREAKS_NEXT_PICKUP`.
   - Same place → 0 minutes. **Equality passes** (`<=`).
6. Return `ok: true` with slacks. `tightFit = slackInMin <= 10` (D-13; assumed, tune on camera).

**Why checking only prev and next is sufficient.** The schedule was valid before insertion. Inserting a ride only changes one predecessor relationship: `next`'s predecessor is now the candidate instead of `prev`. Everything after `next` is unchanged. That's the invariant to explain in the lesson.

**Traffic model note.** Travel time is looked up by **departure time**. A leg that starts in rush hour and ends after it is priced as a rush-hour leg. That's a known simplification; call it out honestly in the episode.

## 6. Late risk: `projectDay(schedule, live, ctx, provider, cfg)`

Called on every driver ping (server side, cached per ping timestamp).

- Inputs: current driver position + time, and which item is active.
- If idle: `projectedArrival(next) = now + provider.minutes(driver.place, next.pickup, now)`.
- If mid-ride: `projectedFree = activeRide.plannedEndAt` (or live-adjusted), then deadhead to next as above.
- `lateMin = max(0, projectedArrival - next.pickupAt)`. Report `AT_RISK` if `lateMin > 0`.
- **Cascade.** A late pickup delays that ride's end by `lateMin`. Propagate: for each following item, `lateMin' = max(0, lateMin - slack(prev→item))`. Flag every item with `lateMin' > 0`. The driver console shows the chain; the async layer notifies riders whose pickups are threatened.
- No auto-reshuffle in MVP (D-26). The engine only reports.

## 7. Slot search: `findSlots({ vehicleId, pickup, dropoff, date, ... })`

**Per car.** The customer flow first asks which cars have at least one slot that day (and seats >= party size), shows each with its price for this trip, then shows the times for the chosen car. Run `findSlots` once per assigned car and cache shared routing calls so the extra cars do not multiply provider cost.

1. Build the day window and the sorted schedule (rides + blocks).
2. Walk the **gaps**. For each gap compute the earliest possible pickup (`prev.endAt + deadhead + buffer`), snap **up** to the 5-min grid, then step forward by `slotStepMin`, calling `canInsert` until the gap's latest possible time is exceeded.
3. Respect `maxProviderCallsPerSearch`; use the routing cache (grid cell × grid cell × hour bucket).
4. Return `Slot[]` with `pickupAt`, `plannedEndAt`, `tightFit`, and a machine-readable `reason` for suppressed times (useful for the "why is 9:00 gone?" tooltip and for teaching).

**"No need to wait an hour" in one line:** slots come from *where the driver will be*, not from a fixed hourly grid.

**Ride now:** `earliestRideNow()` = `canInsert` with `rideNow: true`, starting at `now` from `driverNow`. Returns the first grid-aligned feasible time.

## 8. Test cases (write these before the code)

Fixture places and times: `docs/fixtures/travel-matrix.json`. **Illustrative numbers, not measured traffic.** Demo peak model: departure-time based, peak windows 06:30–09:30 and 15:30–19:00, factor ×1.4 if base < 15 min else ×1.8, rounded up. Config defaults: buffer 5, load 3, unload 2, grid 5, lead 60.

| ID | Scenario | Expected |
|---|---|---|
| **S1** | Empty day, home base in Lewisville. Book `LEW_A → DAL_A` at 08:00 | `ok`. Ride departs 08:03 (peak): 35 × 1.8 = **63 min**. `plannedEndAt = 08:00 + 3 + 63 + 2 = 09:08` |
| **S2** | After S1's ride, earliest new pickup back at `LEW_A` | Return leg departs 09:08 (peak): 63 min → 10:11, +5 buffer = 10:16, snap up → **10:20**. Test a traffic-**blind** engine (flat 35): it would offer **09:20**, 60 min too early. Assert the aware engine rejects 09:20 with `CANT_REACH_PICKUP` |
| **S3** | After S1's ride ends at `DAL_A` 09:08, new ride picking up at `DAL_B` (Deep Ellum, 7 min base) | Deadhead departs 09:08 (peak): 7 × 1.4 → **10 min**, arrive 09:18, +5 = 09:23, snap → **09:25**. Earliest slot is **17 min** after the last drop-off, not an hour. `tightFit = true` |
| **S4** | Schedule: S1 ride (08:00) and a ride at `LEW_A` 10:20. Candidate `DAL_B → PLA` at 09:30 | Prev check ok (09:08 + 10 + 5 = 09:23 ≤ 09:30). Ride departs 09:33 (off-peak): 29 min → ends **10:04** at `PLA`. Return `PLA → LEW_A` 24 min → 10:28, +5 = 10:33 > 10:20 → **`BREAKS_NEXT_PICKUP`, shortfall 13** |
| **S5a** | Candidate pickup 04:50 | `OUTSIDE_AVAILABILITY` |
| **S5b** | `DAL_A → LEW_A` at 23:30 (ends 00:10, after the 24:00 cutoff) | **`ok`**: the ride *starts* before the cutoff (D-09). A ride may finish after the driver's end time |
| **S5c** | Same trip, pickup exactly at the cutoff | `ok` (equality passes). One grid step after the cutoff → `OUTSIDE_AVAILABILITY` |
| **S6a** | `now = 10:00`, driver idle at home. Scheduled pickup at 10:30 | `TOO_SOON`. 11:00 → `ok` if feasible |
| **S6b** | Same `now`, `earliestRideNow()` for `LEW_A` | 10:00 + 6 + 5 = 10:11 → snap → **10:15** |
| **S7** | Empty day. `PLA → FRI` at 05:00 from home | Home → PLA 26 min from 05:00 → 05:26, +5 = 05:31 > 05:00 → `CANT_REACH_PICKUP`, shortfall 31. At **05:35** → `ok` |
| **S8a** | Block at `DAL_A` 12:00–13:00. Candidate `LEW_A → DAL_A` at 11:30 (ends 12:10) | `OVERLAP` |
| **S8b** | Same block. Candidate at 11:15 (ends 11:55 at `DAL_A`) | `ok`: 11:55 + 0 + 5 = 12:00 ≤ 12:00 (**equality passes**) |
| **S9** | Two callers request the same slot concurrently | Integration test: exactly one insert succeeds; the loser re-runs feasibility and gets `OVERLAP` or `CANT_REACH_PICKUP` with fresh alternatives |
| **S11a** | Per-date override: Thursday ends 14:00 (D-44). Candidate `LEW_A → DAL_A` pickup 14:00 (off-peak, ends 14:40) | `ok`. Pickup 14:05 → `OUTSIDE_AVAILABILITY`. The same request on a normal day is unaffected |
| **S11b** | Override closes the whole day | Every candidate → `OUTSIDE_AVAILABILITY`; `findSlots` returns `[]` with reason `DAY_CLOSED` |
| **S11c** | Driver shortens a day that already has a 15:00 booking (D-45) | The existing booking stays valid; `validateSchedule` reports it as `PAST_CUTOFF` (warning, not error); new candidates after the cutoff are rejected |
| **S12a** | Car windows (D-84): Sentra assigned 05:00–13:00, Suburban assigned 13:30–24:00, swap block at `HOME` 13:00–13:30. Candidate pickups: Sentra `LEW_A` 13:35; Suburban `LEW_A` 12:55 | Both `CAR_NOT_ASSIGNED` (wrong car for that time) |
| **S12b** | Same day. Suburban `LEW_A` pickup: 13:35, then 13:45 | 13:35 → `CANT_REACH_PICKUP` (swap block ends 13:30 at `HOME`; 13:30 + 6 + 5 = 13:41 > 13:35, shortfall 6). **13:45 → `ok`** |
| **S12c** | Same day. Sentra ride `LEW_A → DAL_A` at 12:00 (ends 12:40 at `DAL_A`) before the 13:00 swap block at `HOME` | `BREAKS_NEXT_PICKUP`, **shortfall 19**: `DAL_A → HOME` 34 min from 12:40 → 13:14, +5 = 13:19 > 13:00. The engine will not strand him far from the car he must swap |
| **S12d** | Party of 5 requested; Sentra seats 4, Suburban seats 7 | Car list shows only the Suburban (`PARTY_TOO_LARGE` for the Sentra); the Sentra is never offered |
| **S10** | DST days (Sun 2026-03-08 and Sun 2026-11-01) | `dayWindow` returns different UTC offsets but the same 19 h local window; no fixed `-6h` anywhere |
| **L1** | Driver idle at `DAL_A` at 09:50, next pickup `LEW_A` 10:20 | Off-peak 35 → 10:25 > 10:20 → `AT_RISK`, **late 5 min** |
| **L2** | L1 plus a following ride whose slack after the 10:20 ride is 3 min | Cascade: late 5 → following item late `5 - 3 = 2`; both flagged |
| **V1** | `visibilityWindow(pickupAt, now)` | `countdown` if `now < pickupAt - 30m`; `live` inside; `done` after ride end |

### Pricing (same fixture plus `docs/fixtures/vehicles.json`, `lib/pricing`)

A price configuration belongs to a **car** (D-80/D-81). Tiers are evaluated on the **off-peak reference trip time**, integer cents, first tier with `upToMinutes >= minutes` wins (`null` = unbounded). Feasibility does not depend on the car (D-83); only price and display do.

| ID | Trip / setup | Expected |
|---|---|---|
| P1 | `LEW_A → DAL_A`, off-peak reference 35 min, car 1 (`pc_std_v1`) | **$25** (matches "Lewisville → Dallas, $25", D-08) |
| P2 | `DAL_A → DAL_B`, 7 min, car 1 | **$20** |
| P3 | `LEW_A → DAL_A` quoted at 08:00 vs 14:00, car 1 | Both **$25**. Price ignores traffic |
| P4 | `FRI → FTW_A`, reference 52 min, car 1 | **$30** |
| P5 | Out-of-area point | No quote; `OUT_OF_AREA` |
| P6 | Same `LEW_A → DAL_A` trip priced for both cars in the customer's car list: car 1 (`pc_std_v1`) and car 2 (`pc_lux_v1`) | Car 1 **$25**, car 2 **$85** (its 30–45 tier; placeholder Black-style ladder, D-123). Where both cars are assigned to the same time the offered pickup times are identical (feasibility is car-independent); the customer sees both prices side by side |
| P7 | Book at `pc_std_v1` v1 ($25), then the owner publishes v2 with $28 for that tier | Existing booking still **$25** (snapshot of config id + version). A new quote is **$28**. Boundary: exactly 30 min → first tier; exactly 45 → second |

### Policy (`lib/policy`)

| ID | Case | Expected refund |
|---|---|---|
Policy v1 (D-52, D-53, D-55), modeled on Uber/Lyft. Fare $20 for these rows. Amounts are config in `policy.config.ts`.

| ID | Case | Rider is charged | Refund |
|---|---|---|---|
| C1 | Rider cancels at T-90 min | $0 | $20.00 |
| C2 | Rider cancels at T-30 min, driver not yet en route | $5 fee | $15.00 |
| C2b | Rider cancels after the driver tapped `Heading to pickup` | $10 fee | $10.00 |
| C3 | Driver cancels at any time | $0 | $20.00 |
| C4 | No-show marked ≥ 5 min after `Arrived` (rider was contacted) | $10 fee | $10.00 |
| C5 | No-show marked 3 min after `Arrived` | Rejected: grace not elapsed | n/a |
| C6 | Projected driver lateness ≥ 10 min, rider cancels | $0 | $20.00 |
| C7 | Fee is capped at the fare paid (fare $8 test config, fee $10) | fare only | $0 |

## 9. Property tests (the ones that impress)

Generate random valid days with a seeded PRNG, then:

- **Soundness:** every slot from `findSlots` → `canInsert` ok → `validateSchedule` on the new day is ok.
- **Completeness (on the grid):** every grid time *not* returned fails `canInsert`.
- **Monotone insertion:** inserting a valid ride never invalidates any pre-existing item.
- **Determinism:** same inputs, same outputs (no clocks, no randomness inside).
- **Provider budget:** `findSlots` never exceeds `maxProviderCallsPerSearch` provider calls with a cold cache.

## 10. Honest limitations (say these on camera)

1. Departure-time traffic lookup; a leg crossing the end of rush hour is approximated.
2. Predictive traffic is an estimate. Buffer and peak factors need calibration from real trips after launch.
3. Drivers get late for reasons no model sees (a rider takes 8 minutes to walk out). Dwell times are config, not truth.
4. One driver, one vehicle. Multi-driver is a different problem (assignment), not this one.

## 11. Generalizing (consulting door)

Rename `ride` → `job`, `pickup/dropoff` → `start/end place`, and the same module schedules a mobile detailer, notary, groomer, or barber. Keep a `docs/CASE_STUDY.md` section, "Same engine, different business."
