# GBRSA Records

World (**WR**) and Asian (**AS**) speed-event records, served as static JSON and consumed at runtime by
**Gbrsa SupaApp**.

This repository is the **single source of truth** for those records. The app fetches `records.json` from
a CDN when it exports a results PDF, so **updating a record is just a commit here** — no app rebuild, no
redeploy.

![CDN](https://img.shields.io/badge/CDN-jsDelivr-yellow)

## Live endpoint

```
https://cdn.jsdelivr.net/gh/weitian1993gbrsa/gbrsa-records@main/records.json
```

Open that URL in a browser and you should see the JSON. The app points at the same URL
(`RECORDS_CDN_URL` in `src/services/records/recordsService.ts`).

## What's in here

| File | Purpose |
| --- | --- |
| `records.json` | The data. Every record row lives here. |
| `README.md` | This file. |

`records.json` is wrapped in a small envelope:

```json
{
  "version": 1,
  "updated": "2026-10-13",
  "source": "IJRU world records (...) + AJRU Asian records (...)",
  "records": [ /* ... rows ... */ ]
}
```

Only `records` is required by the app; `version`, `updated` and `source` are bookkeeping/metadata.

## Row format

One row per `event + division + type`:

```json
{
  "event": "SRSS",                    // event CODE as used in the app (see below)
  "division": "OPEN FEMALE",          // division name as used in the app (matched case-insensitively)
  "type": "WR",                       // "WR" = world, "AS" = Asian; printed verbatim in the Record column
  "result": "30.12",                  // printed verbatim (seconds for speed events)
  "holder": "JANE DOE",               // athlete or team name
  "country": "MAS",                   // optional — NOC code column
  "location": "BANGKOK (THA)",        // optional — where it was set (falls back to `meet`)
  "date": "2026-08-01",               // optional — ISO date, rendered as "01 AUG 2026"; free text passes through
  "meet": "World Championships 2026"  // optional — event/competition name
}
```

### Fields

| Field | Required | Notes |
| --- | --- | --- |
| `event` | ✅ | Event code, e.g. `SRSS`. Matched case/spacing-insensitively. |
| `division` | ✅ | e.g. `OPEN FEMALE`. Matched case/spacing-insensitively. |
| `type` | ✅ | `WR` or `AS` (any short code works; it is printed as-is). |
| `result` | ✅ | Printed verbatim in the table (e.g. `429 (TIE)`). |
| `holder` | ✅ | Athlete or team name. |
| `country` | — | NOC/country code, e.g. `HKG`. |
| `location` | — | Where it was set; falls back to `meet` if omitted. |
| `date` | — | ISO `YYYY-MM-DD` → rendered `DD MON YYYY`; any other string is shown as-is. |
| `meet` | — | Competition name. |

## Event codes

These are the speed events currently covered (codes match `EVENTS_CATALOG` in the app):

| Code | Event |
| --- | --- |
| `SRSS` | Single Rope Speed Sprint |
| `SRSE` | Single Rope Speed Endurance |
| `SRDR` | Single Rope Double Unders Relay |
| `SRSR` | Single Rope Speed Relay |
| `SRTU` | Single Rope Triple Unders |
| `DDSS` | Double Dutch Speed Sprint |
| `DDSR` | Double Dutch Speed Relay |

Divisions currently seeded: `OPEN FEMALE`, `OPEN MALE`, `OPEN MIXED`.

## How the app uses it

When a results PDF is exported, the app looks up the record rows for each event code + division and
renders a small table above the results:

> **Record · Result · Name · NOC Code · Location · Date**

Rows are ordered `WR` → `AS`. The JSON is fetched once per session, cached in `localStorage` with a
6-hour TTL, and falls back to the last good copy if the network is unavailable — so exporting never
fails because the CDN is down.

## How to add or change a record

1. Edit `records.json` (the GitHub web UI is fine — click ✏️ → commit).
2. That's it. The app picks it up on its next fetch.

**Caching:** `@main` on jsDelivr can lag a few minutes behind a commit. If you need a change live
instantly, point at a specific commit or tag instead of `@main` (e.g.
`.../gbrsa-records@<commit-sha>/records.json`).

### Rules

- One row per `event + division + type`. A newer row for the same key supersedes the old one.
- Divisions and event codes are matched **case-insensitively** with extra spaces collapsed, so
  `"OPEN FEMALE"` and `"open  female"` are the same.
- If a `WR` row is missing for a division, only the `AS` row (or nothing) is shown — the table adapts.

## Data sources

- **IJRU World Records** — <https://ijru.sport/world-records>
- **AJRU Asian Records** — <https://www.ajru.sport/asian-records>

Records are mapped to the OPEN divisions of the speed events. Where an Asian athlete or team also holds
the world record, the `WR` and `AS` rows are identical (as in the source data).

## Used by

- **Gbrsa SupaApp** (private repo) — reads this JSON at PDF-export time via
  `src/services/records/recordsService.ts`.
