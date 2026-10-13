# GBRSA Records

World (WR) and Asian (AR) records for the speed events, consumed by **Gbrsa SupaApp**.

This repo is the **single source of truth**. The app fetches `records.json` at runtime, so updating a
record is a **commit here** — no app rebuild or redeploy needed.

## How to add / change a record

1. Edit **`records.json`** (GitHub web UI is fine — click ✏️ → commit).
2. Done. The app picks it up (jsDelivr may cache for a few minutes; change the URL to a tag/commit hash
   if you need it instantly).

## Row format

```json
{
  "event": "SRSS",                    // event CODE as used in the app: NESS, SRSS, SRDR, SRSR, SRTU, DDSS, DDSR
  "division": "U12- FEMALE",          // division name EXACTLY as in the app (case-insensitive match)
  "type": "WR",                       // "WR" world, "AR" Asian (WJ/AS/GR also allowed) — printed in the Record column
  "result": "30.12",                  // printed verbatim (seconds for speed events)
  "holder": "JANE DOE",               // athlete or team name
  "country": "MAS",                   // optional — NOC code column
  "location": "BANGKOK (THA)",        // optional — where it was set (falls back to `meet`)
  "date": "2026-08-01",               // optional — ISO date, rendered as "01 AUG 2026"
  "meet": "World Championships 2026"  // optional
}
```

The PDF prints these as a small table above the results (**Record · Result · Name · NOC Code ·
Location · Date**), ordered WR → AR → others.

### Example

```json
{
  "version": 1,
  "updated": "2026-08-02",
  "records": [
    {
      "event": "SRSS",
      "division": "U12- FEMALE",
      "type": "WR",
      "result": "30.12",
      "holder": "JANE DOE",
      "country": "MAS",
      "location": "BANGKOK (THA)",
      "date": "2026-08-01"
    },
    {
      "event": "SRSS",
      "division": "U12- FEMALE",
      "type": "AR",
      "result": "31.40",
      "holder": "MEI LING",
      "country": "CHN",
      "location": "JINAN (CHN)",
      "date": "2025-11-20"
    }
  ]
}
```

## Rules

- One row per `event + division + type`. A newer row for the same key replaces the old one.
- Divisions are matched case-insensitively with extra spaces collapsed, so `"U12- FEMALE"` and
  `"u12-  female"` are the same.
- `type` is printed as-is; order in the table is `WR` then `AR` then anything else.
