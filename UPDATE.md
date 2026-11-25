# Updating the public Dehradun traffic site

The live site is a static GitHub Pages site.

- **Public URL:** https://amanrubey.github.io/dehradun-traffic/
- **Repository:** https://github.com/amanrubey/dehradun-traffic
- **Account:** `amanrubey` (user-GitHub-xai connector)
- **Branch:** `main`
- **Files that matter:** `index.html` (page and charts) and `data.json` (all readings). Charts, averages, best/worst, and the readings table are computed in the browser from `data.json`, so most updates only need a data change.

`gh` is not authenticated on the box. Do not use surge.sh (it needs a login). Publish with the already-connected **user-GitHub-xai** MCP tools.

## 1. Append the new reading locally

Work in `/workspace/traffic-site/` (keep that folder name).

Open `data.json` and:

1. Append one object to `readings` for a 9 AM / 1 PM / 4 PM / 6 PM outbound check, using the existing shape:

```json
{
  "date": "2026-11-25",
  "weekday": "Wed",
  "dayType": "Weekday",
  "slot": "evening",
  "gms": 9,
  "clock": 18,
  "pacific": 16,
  "centrio": 19,
  "himadri": 54,
  "notes": "Optional route notes. Use null, not a guess, for any route that was not recorded."
}
```

- `slot` must be one of: `morning` (~9 AM), `afternoon` (~1 PM), `leave` (~4 PM), `evening` (~6 PM). The page picks up new slot keys automatically if you also add a label under `slotMeta`.
- `dayType` is `Weekday`, `Saturday`, or `Sunday`.
- Times are **minutes as numbers**. Never invent a time; use `null` when a route was not recorded.
- For a Himadri → Ballupur return check, append to `returns` instead:

```json
{
  "date": "2026-11-25",
  "weekday": "Wed",
  "timeLabel": "around 7:10 PM IST",
  "minutes": 46,
  "via": "Raipur Rd",
  "notes": "Optional alternatives and conditions."
}
```

2. Set `lastUpdated` to the current time in IST, for example `25 Nov 2026, 6:15 PM IST`.
3. Set `latestReading` to a short label for the newest check, for example `Wed 25 Nov 2026 evening, around 6:15 PM IST`.
4. Update `gaps` only when a known hole is filled or a new one appears. Refresh `alternatives` and any hand-written notes only when the new check changes those conclusions. Do not edit `index.html` unless the page itself needs a change.

Keep a copy of the updated `data.json` (and `index.html` if you changed it) in `/workspace/traffic-site/` as well as in the repo.

## 2. Republish to GitHub Pages

Use the **user-GitHub-xai** connector (GetDynamicTools first if the schema is needed):

1. Call `get_file_contents` with `owner`: `amanrubey`, `repo`: `dehradun-traffic`, `path`: `data.json`, `ref`: `main`. Note the file’s blob SHA.
2. Call `create_or_update_file` with:
   - `owner`: `amanrubey`
   - `repo`: `dehradun-traffic`
   - `path`: `data.json`
   - `branch`: `main`
   - `sha`: the SHA from step 1 (required because the file already exists)
   - `message`: a short commit message such as `Add 25 Nov 2026 evening readings`
   - `content`: the full new `data.json` text (plain text, not base64)
3. If `index.html` (or another file) also changed, repeat get-SHA then `create_or_update_file` for that path, or use `push_files` to commit several files in one go (`owner`, `repo`, `branch`: `main`, `files`: `[{path, content}, ...]`, `message`).
4. Pages usually updates in about a minute. Verify with:

```bash
curl -sI https://amanrubey.github.io/dehradun-traffic/
curl -s https://amanrubey.github.io/dehradun-traffic/data.json | head
```

Expect HTTP 200 and the new `lastUpdated` value. Retry a few times if it still shows the old file (CDN/Pages lag). Adding `?v=YYYYMMDDHHMM` is not required; the page fetches `data.json` with `cache: "no-store"`.

Do not change Pages settings unless the site is down. It is served from the `main` branch, site root (`/`), on this public repository.
