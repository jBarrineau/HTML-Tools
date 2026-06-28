# Build Plan — Power Automate Recurrence Schedule Builder

A standalone HTML tool that lets a developer configure a Power Automate (Logic Apps) **Recurrence** trigger visually, see exactly when it will fire, and copy a correct `recurrence` JSON object into the trigger's code view.

This document is a build spec for a coding agent. It assumes no prior conversation context.

---

## 1. Goal & guiding principle

The Power Automate maker portal lets you configure a recurrence trigger but gives **almost no feedback about when the flow will actually run**. Misconfigurations (wrong run count, timezone drift, ignored schedule fields) only surface in production.

**The killer feature is a "next N runs" preview.** The tool computes the next 10–15 actual fire times from the current configuration and displays them in both the flow's configured timezone and the user's local time, side by side. Every other part of the tool exists to feed that preview. When a user sees the projected timestamps, they catch their own mistakes immediately.

Design priority order:
1. Correct fire-time computation (the preview must be trustworthy).
2. Correct `recurrence` JSON output.
3. A warnings panel that explains *why* something is wrong.
4. Pleasant, fast UI.

---

## 2. Constraints & tech choices

- **Single self-contained `.html` file.** No build step, no bundler. This matches the existing suite of Power Platform HTML tools and keeps it trivially shareable.
- **Vanilla JS + plain CSS.** No framework required. If the agent prefers, a single `<script type="module">` is fine, but no npm/build pipeline.
- **No network calls at runtime.** All timezone data is bundled or derived from the browser's `Intl` API.
- **Target modern evergreen browsers** (Chromium/Firefox/Safari current). This lets us rely on `Intl.DateTimeFormat` with `timeZone`, and `Intl.supportedValuesOf('timeZone')`.

---

## 3. Domain background the agent must understand

Power Automate's recurrence is the Azure Logic Apps recurrence engine. The `recurrence` object has these fields:

| Field | Type | Notes |
|---|---|---|
| `frequency` | string | One of `Second`, `Minute`, `Hour`, `Day`, `Week`, `Month`. |
| `interval` | int | Every N of the frequency unit. |
| `timeZone` | string | A **Windows** timezone ID, e.g. `"W. Europe Standard Time"` — **not** an IANA name like `Europe/Berlin`. |
| `startTime` | string | ISO-ish local datetime, e.g. `"2026-01-05T09:00:00"`. Acts as an anchor / "don't run before" boundary. Does **not** backfill missed runs. |
| `schedule` | object | Only meaningful for `Day` and `Week`. Refines *when within the cadence*. |
| `schedule.hours` | int[] | Hours of day, 0–23. |
| `schedule.minutes` | int[] | Minutes, 0–59. |
| `schedule.weekDays` | string[] | `"Monday"`…`"Sunday"`. Only used by `Week` frequency. |

### Semantics that drive the logic

- **hours × minutes is a cross-product.** `hours: [9, 17]` with `minutes: [0, 30]` produces **four** runs per day: 09:00, 09:30, 17:00, 17:30 — not two. This is the #1 user surprise.
- **`schedule` only applies to `Day` and `Week`.** For `Second`/`Minute`/`Hour`/`Month`, the schedule's hours/minutes/weekDays are ignored. `Week` additionally filters by `weekDays`; `Day` ignores `weekDays`.
- **When a `schedule` is present, the time-of-day comes from the schedule**, not from `startTime`. `startTime` mostly acts as the earliest-allowed boundary and date anchor. Users who set `startTime` 14:00 with `hours:[9]` get 9:00 and are confused.
- **`startTime` in the past does not replay missed runs.** Projection is forward from "now".
- **Timezone & DST.** If `timeZone` is set, the runtime honors wall-clock across DST transitions (a 9am job stays 9am). If only a `startTime` with a UTC offset is given and no `timeZone`, the time drifts by an hour at each DST boundary.
- **Minimum interval is plan-dependent**, so we warn softly rather than hard-block tight intervals.
- **Month-level "run on the 1st" is a weak spot.** The schedule object doesn't cleanly express day-of-month, so we detect that intent and suggest the common workaround rather than pretending it works.

---

## 4. UI layout

Three regions, ideally a two-column layout collapsing to one column on narrow screens.

**Left column — Configuration**
1. Frequency selector (segmented control or dropdown) + interval number input ("Every `[ N ]` `[ Week ]`").
2. Timezone picker — searchable dropdown. Displays friendly label **with current UTC offset** (e.g. "(UTC+01:00) W. Europe — Berlin, Paris"). Emits the Windows ID internally. See §6.
3. Start time — date+time inputs (and an optional "no start time" state).
4. Schedule controls (conditionally enabled — see below):
   - Weekday toggle row: M T W T F S S (multi-select).
   - Hours grid: 0–23, multi-select (a compact 6×4 or 12×2 grid).
   - Minutes selector: multi-select chips for common values (0, 15, 30, 45) plus an "add custom minute" affordance for 0–59.

**Conditional enabling:** when frequency is not `Day` or `Week`, grey out the schedule controls and show an inline note ("Schedule applies only to Day and Week frequency — these settings will be ignored"). When frequency is `Day`, grey out weekDays with a note ("weekDays applies only to Week frequency").

**Right column — Output & feedback**
1. **Next runs preview** (the centerpiece): a table of the next ~12 fire times, two columns — *Flow timezone* and *Your local time* — plus a relative hint ("in 3 hours"). Prominent **run-count summary**: "This configuration produces **4 runs per day**."
2. **Warnings panel** (see §7): a list of active warnings, each with a one-line explanation.
3. **Plain-English restatement**: "Runs every weekday at 9:00 and 9:30, W. Europe Standard Time."
4. **JSON output**: the `recurrence` object in a readonly code block with a Copy button.

Everything is live: any input change recomputes preview, warnings, English summary, and JSON.

---

## 5. Core algorithm — computing the next N fire times

This is the hardest and most important part. Get it right before polishing UI.

### 5.1 Inputs to the function

```
computeNextRuns({
  frequency, interval, ianaTimeZone, startTime /* may be null */,
  hours, minutes, weekDays, count /* e.g. 12 */, now /* Date */
}) -> Array<Date /* UTC instants */>
```

Note: internally work in the **IANA** zone (mapped from the chosen Windows ID, see §6) because that's what `Intl` understands.

### 5.2 Two regimes

**Regime A — schedule-driven (`Day` and `Week`):**
The fire times are wall-clock times in the target zone defined by the cross-product of `hours × minutes`, on qualifying days.

Algorithm:
1. Default `minutes` to `[0]` if empty but `hours` is set. Default `hours`/`minutes` sensibly if both empty (e.g. derive from `startTime`'s time-of-day, or `[0]`/`[0]`).
2. Let `cursorDate` = the later of `today` and `startTime`'s date, in the target zone.
3. Iterate forward day by day. For each day:
   - For `Week` frequency: skip the day if its weekday is not in `weekDays` (and apply the `interval` week-stride relative to the `startTime` week — every Nth week).
   - For `Day` frequency: apply the `interval` day-stride relative to `startTime`'s date (every Nth day).
   - For each `h` in sorted `hours`, each `m` in sorted `minutes`: construct the wall-clock instant `{thatDay, h, m, 0}` **in the target zone**, convert to a UTC instant (§5.4), and if it is `>= now` and `>= startTime`, collect it.
4. Stop once `count` instants are collected (guard with a max-iterations cap, e.g. 366×something, to avoid infinite loops when `weekDays` is empty for a Week trigger — treat empty weekDays as a warning + no runs).

**Regime B — interval-driven (`Second`, `Minute`, `Hour`, `Month`):**
There is no meaningful schedule. Fire times are `anchor + k·interval·unit`.
1. `anchor` = `startTime` if present, else `now`.
2. Generate `anchor, anchor+interval·unit, anchor+2·interval·unit, …`, advance until `>= now`, then collect `count` of them.
3. For `Month`, add calendar months (not 30 days) — use zone-aware month arithmetic. Preserve day-of-month from the anchor; clamp for short months.

### 5.3 Cross-product run count

Compute `runsPerDay = max(1, hours.length) × max(1, minutes.length)` for Day/Week regimes and surface it prominently. This number, shown before the user even reads the preview, is the single most effective teaching element.

### 5.4 Wall-clock → UTC instant in a named zone (the tricky bit)

JS `Date` has no "construct a date in zone X" primitive. Implement a helper:

```
function zonedWallClockToUtc(year, month, day, hour, minute, ianaZone) -> Date
```

Recommended approach: build a guess UTC date, format it back into the target zone with `Intl.DateTimeFormat(..., { timeZone: ianaZone, ... })`, compute the offset between the guess and what the zone shows, and correct. Do one correction pass, then re-verify (two passes handles DST-boundary edge cases). This avoids bundling a tz database.

DST edge cases to handle explicitly:
- **Spring-forward gap** (e.g. 02:30 doesn't exist): decide a policy (skip the run, or roll forward to the next valid instant) and document it. Logic Apps' real behavior here is subtle; the tool should at minimum not crash and should note the ambiguity.
- **Fall-back overlap** (e.g. 02:30 occurs twice): pick the first occurrence; note it.

These are rare but the dual-timezone preview will expose them, so handle gracefully.

### 5.5 Output formatting

Format each instant twice via `Intl.DateTimeFormat`:
- Flow timezone: `timeZone: ianaZone`.
- Local: no `timeZone` (browser default), label it with the resolved local zone name.
Include weekday and a relative "in X" hint.

---

## 6. Timezone picker & Windows↔IANA mapping

This is a second meaningful chunk of work and a genuine reason the tool is useful — the Windows timezone list is annoying to assemble correctly.

Requirements:
- The dropdown is **searchable** and shows friendly labels with **live UTC offset** (computed for "now" via `Intl`, so it reflects current DST).
- Internally each option carries `{ windowsId, ianaZone, label }`.
- The JSON output emits `windowsId`. The preview computation uses `ianaZone`.

Implementation:
- Bundle a curated **Windows → IANA** map derived from the CLDR `windowsZones` mapping. A full map is ~140 entries; a curated subset covering common business zones (Americas, Europe, Asia-Pacific majors) is acceptable for v1, with a note in the source on how to extend it. Include the full map if convenient — it's static data.
- For each entry, compute the current offset at load time with `Intl.DateTimeFormat('en', { timeZone: iana, timeZoneName: 'shortOffset' })` and use it in the label.
- Provide a defensive check: if a `timeZone` string the tool produces ever *looks* IANA (`contains "/"`), that's a bug guard — but more importantly, if you add an "import existing JSON" feature (stretch goal), warn when a pasted `timeZone` is IANA-looking, since that's a real user mistake.

---

## 7. Warnings panel — the teaching layer

Each rule below renders as a dismissible-but-recomputing warning with a short explanation. This is what turns the tool from a JSON emitter into something that prevents tickets.

1. **High run count from cross-product.** When `runsPerDay > 1`, state it plainly: "hours × minutes produces N runs per day (cross-product, not a list)." Escalate tone if `runsPerDay` is large (e.g. > 8).
2. **Schedule ignored for this frequency.** If the user has set hours/minutes/weekDays but frequency ∉ {Day, Week}: "These schedule settings are ignored for `<frequency>` frequency."
3. **weekDays ignored on Day frequency.** If `Day` and `weekDays` non-empty.
4. **Empty weekDays on Week frequency.** If `Week` and `weekDays` empty: "No weekdays selected — this trigger will never fire." (And the preview will be empty; make sure §5.2 step 4's cap handles this.)
5. **No timezone set.** If `timeZone` is empty: "Without a timezone, fire times drift by an hour at each DST transition. Set an explicit timezone." Strongly recommended default: pre-select the user's mapped local zone on load.
6. **Past start time.** If `startTime` < now: "Start time is in the past. Missed runs are **not** replayed; the schedule projects forward from now."
7. **startTime time-of-day conflicts with schedule.** If a schedule is present and `startTime`'s HH:MM is not in the hours×minutes set: "The schedule governs the time of day; `startTime`'s time (`HH:MM`) is used only as a boundary, not as a run time."
8. **Empty minutes defaulted.** If hours set and minutes empty: "Minutes defaulted to `[0]`."
9. **Very tight interval.** If frequency is `Minute`/`Second` with a small interval: "Minimum interval is plan-dependent; tight intervals may be throttled to a higher floor on your license."
10. **Month day-of-month intent.** If frequency is `Month`: "The schedule object can't reliably express 'on the Nth of the month.' Consider a daily trigger with a `dayOfMonth` trigger condition / `if(equals(dayOfMonth(...),1), ...)` guard instead." (Provide the workaround snippet.)

---

## 8. Output: JSON + English

**JSON** — emit only the fields that apply to the chosen frequency. Don't emit a `schedule` block for non-Day/Week frequencies. Example for a weekday morning job:

```json
"recurrence": {
  "frequency": "Week",
  "interval": 1,
  "timeZone": "W. Europe Standard Time",
  "startTime": "2026-01-05T09:00:00",
  "schedule": {
    "weekDays": ["Monday","Tuesday","Wednesday","Thursday","Friday"],
    "hours": [9],
    "minutes": [0, 30]
  }
}
```

Provide a Copy button. Consider a toggle for "wrapped in `recurrence`" vs "bare object," since pasting location varies.

**Plain-English restatement** — generate from the config, e.g. "Runs every weekday at 9:00 and 9:30 (W. Europe Standard Time), starting 5 Jan 2026." Keep this generation rule-based and covered by tests.

---

## 9. Test cases (build these as assertions; they pin the tricky behavior)

The agent should implement a small in-page test harness (or comment block of expected values) for at least:

1. **Cross-product count.** Day, hours `[9,17]`, minutes `[0,30]` → 4 runs/day; first four times correct in order.
2. **Week with weekDays filter.** Week, weekDays Mon–Fri, hours `[9]`, minutes `[0]` → next run skips weekend correctly.
3. **Interval stride.** Day, interval 2 → runs every other day from anchor.
4. **DST spring-forward.** A daily 02:30 job across a spring-forward date in a DST zone → documented policy applied, no crash, preview still advances.
5. **DST fall-back.** Daily 02:30 across fall-back → first occurrence chosen.
6. **Past startTime.** No backfill; first run is in the future.
7. **Empty weekDays on Week.** Zero runs + warning #4 fires.
8. **Minute frequency.** Minute, interval 15 → 15-minute spacing, schedule ignored, warning #2 if schedule set.
9. **Month stride.** Month, interval 1, anchor on the 31st → clamps correctly for short months.
10. **Timezone offset label** reflects current DST for a sample zone.

---

## 10. Suggested build phases

**Phase 1 — Skeleton & state.** Single HTML file, layout regions, a central config object, live re-render on input. No real computation yet (stub the preview).

**Phase 2 — Timezone data & picker.** Bundle the Windows↔IANA map, build the searchable offset-labelled dropdown, default to local zone. Implement `zonedWallClockToUtc` (§5.4) and unit-test it against known offsets.

**Phase 3 — Fire-time engine.** Implement `computeNextRuns` for both regimes (§5). Wire the preview table (dual timezone). Implement the test harness (§9) and make it pass — do not move on until DST cases behave.

**Phase 4 — Warnings + English + JSON.** Implement all §7 rules, the §8 JSON emitter (frequency-aware field omission), and the plain-English generator.

**Phase 5 — Polish.** Conditional enabling/greying of schedule controls, run-count prominence, copy buttons, responsive collapse, empty/error states.

**Phase 6 — Stretch (optional).** "Import existing recurrence JSON" to reverse-populate the UI (with the IANA-timezone-mistake warning); preset buttons ("Every weekday 9am", "Hourly during business hours", "1st of month workaround"); a shareable URL that encodes the config in the query string.

---

## 11. Definition of done

- Changing any input live-updates preview, run count, warnings, English summary, and JSON.
- The next-runs preview is correct for all §9 test cases, including DST.
- Timezone dropdown emits Windows IDs, computes preview via IANA, shows live offsets.
- JSON omits inapplicable fields and copies cleanly into a trigger code view.
- All §7 warnings fire under their conditions.
- Single file, opens with a double-click, no console errors, no network calls.
