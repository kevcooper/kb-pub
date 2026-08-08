# Source — DEF CON 34 schedule

## Where it comes from

<https://info.defcon.org/> — the official DEF CON 34 information app. The data
was originally captured by hand from that site (a content-cards index plus
paginated content detail records), then flattened to `schedule.csv`.

Pulled: 2026-08-08, during the conference.

## Shape

`schedule.csv` — one row per **session**, not per talk. A talk given twice has
two rows sharing a `content_id` but with different `session_id`s. `session_id`
is the identifier every tool uses, and what `picks.json` references.

| column | notes |
| --- | --- |
| `content_id` | the talk/activity |
| `session_id` | this specific occurrence — the id you `pick` |
| `day` | e.g. `Sat 2026-08-08` |
| `begin_local` / `end_local` | conference-local, Las Vegas (UTC-7 in August) |
| `begin_utc` / `end_utc` | same instants in UTC |
| `tags` | `;`-separated, e.g. `DEF CON Official Talk; Exploit 🪲` |
| `links` | `;`-separated `Label: URL` pairs — the materials harvest reads this |

Scale as captured: 1,848 sessions, 171 distinct tags, 170 distinct locations.
Fri (739) and Sat (717) are by far the heaviest days; 553 sessions carry links
and 1,263 name speakers. Rows also exist for Wed–Thu pre-con and Mon–Tue
post-con training.

## Quirks

- One row has an empty `day` — filtered out on load.
- Tag vocabulary is not controlled: `DEF CON Workshop` and `DEF CON Workshops`
  both occur, and some tags carry emoji (`Demo 💻`, `Tool 🛠`). Match tags
  case-insensitively and by substring, which is what `dc.py --tag` does.
- Location strings encode building, level and room, e.g.
  `LVCC - L1 - Exhibit Hall West 3 - 904 (Main Track 4)`. `dc.py` takes the
  leading segment as the venue to spot cross-campus hops, and the parenthesised
  tail as the human-readable room.
- Mid-conference changes happen: rooms move and talks get cancelled. Re-run the
  refresh rather than hand-editing.

## Refresh

```bash
python3 tools/fetch_defcon.py --dry-run
python3 tools/fetch_defcon.py
```

See `.claude/skills/refresh-defcon-schedule/`. The endpoint paths in
`tools/fetch_defcon.py` (`BASE`, `ENDPOINTS`) are a best guess at the app's API
and are **unverified against the live site** — the original capture was manual.
If the fetch fails, use `--from-dir <export>` to re-flatten a manual export,
then fix the endpoints and record what changed here.

## Terms

Conference schedule data, used personally to plan attendance. Not
redistributed. Archived materials under `materials/` belong to their
respective authors — that directory is gitignored by default.
