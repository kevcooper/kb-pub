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

See `.claude/skills/refresh-defcon-schedule/`.

### Upstream layout (verified)

The site is built from <https://github.com/junctor/hackertracker-info> and is a
static JSON tree, not a query API. The layout is declared in that repo:

- `src/lib/conferences.ts` — `dataRoot` per conference; DEF CON 34 is `/ht/defcon34`
- `src/lib/dataContract.ts` — view paths, detail shard counts, `HT_SCHEMA_VERSION = 4`

Upstream publishes a **ready-made schedule export**. It is NOT used by default,
because it is lossy — verified against the live file on 2026-08-08, it has no
`links` column (losing materials for 553 sessions), truncates abstracts to a
`description_snippet`, carries UTC only, and yields 1,847 rows against 1,848
from the detail shards. Use it only when titles and times are all you need:

| what | url |
| --- | --- |
| schedule export (preferred) | `https://info.defcon.org/ht/defcon34/exports/schedule.csv` |
| same, as JSON | `https://info.defcon.org/ht/defcon34/exports/schedule.json` |

Its columns are `session_id, content_id, start_utc, end_utc, location_name,
title, speaker_names, organization_names, tag_names, description_snippet` —
which is *not* the shape the original hand capture had, so the capture came
from somewhere richer.

The detail shards are the default source and produce the full record:

| what | url |
| --- | --- |
| manifest | `https://info.defcon.org/ht/defcon34/manifest.json` |
| content cards | `https://info.defcon.org/ht/defcon34/views/contentCards.json` |
| content detail | `https://info.defcon.org/ht/defcon34/details/content/{00..NN}.json` |

No shard count is hardcoded. `shardCount` is a build parameter upstream, not
covered by `schemaVersion`, and it differs per conference — a training event
has far less content than DEF CON itself. The range is discovered by walking
from zero until two consecutive misses, and completeness is settled against
`contentCards.json`, which enumerates every content record.

Note the host serves a SPA, so a missing path returns `index.html` with a
**200**, not a 404 — the end of the range shows up as a JSON parse failure.
That is why a single failure does not end the walk: it could equally be one
flaky request, and a success after it resets the count.

Those eight shards are exactly the eight JSON files in the original hand
capture. Locally they are stored as `content/page-NN.json`, where `NN` matches
the upstream shard number.

`manifest.json` returns `{buildTimestamp, schemaVersion}`. The refresh script
checks `schemaVersion` and warns if it is no longer 4 — that is the cheap
signal that the layout changed and `dataContract.ts` needs re-reading.

A different year needs no code change: `--conference defcon35 --out data/defcon35`.

### The host is HTTP/2-only

It resets HTTP/1.1 connections, so Python's `urllib` cannot reach it at all —
the symptom is `[Errno 54] Connection reset by peer` after a successful TLS
handshake, which looks like bot-blocking and is not. `tools/fetchlib.py` shells
out to `curl` (which negotiates h2 over ALPN) whenever it is installed. If a
refresh fails with resets, check `curl --http1.1` vs `--http2` before touching
headers; `python3 tools/diagnose_fetch.py` does it for you.

## Terms

Conference schedule data, used personally to plan attendance. Not
redistributed. Archived materials under `materials/` belong to their
respective authors — that directory is gitignored by default.
