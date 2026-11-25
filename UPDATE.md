# Updating the public Dehradun traffic site

Source of truth is the public GitHub repo. The shareable site is served from the `main` branch.

- **Public URL (working now):** https://raw.githack.com/amanrubey/dehradun-traffic/main/index.html
- **Intended GitHub Pages URL (not enabled yet):** https://amanrubey.github.io/dehradun-traffic/
- **Repository:** https://github.com/amanrubey/dehradun-traffic
- **Account:** `amanrubey` (user-GitHub-xai connector)
- **Branch:** `main`
- **Files that matter:** `index.html` (page and charts) and `data.json` (all readings). Charts, averages, best/worst, and the readings table are computed in the browser from `data.json`, so most updates only need a data change.

`gh` is not logged in on the box. Do not use surge.sh (it needs a login). Publish with the already-connected **user-GitHub-xai** MCP tools. The GitHub connector has no Pages API, and a GitHub Actions `GITHUB_TOKEN` cannot enable Pages, so official GitHub Pages is not live until someone turns it on once in the repo (Settings → Pages → Source: GitHub Actions). A deploy workflow is already in `.github/workflows/pages.yml`. Until that switch is flipped, share the raw.githack URL above (verified HTTP 200, `text/html`, with `data.json` on the same origin).

## 1. Append the new reading locally

Work in `/workspace/traffic-site/` (keep that folder name). Also keep a copy in `/workspace/traffic-site-backup/` if that folder is present.

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

## 2. Republish

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
3. If `index.html` (or another file) also changed, repeat get-SHA then `create_or_update_file` for that path, or use `push_files` to commit several files in one go (`owner`, `repo`, `branch`: `main`, `files`: `[{path, content}, ...]`, `message`). `push_files` allows at most four `$file:` path substitutions per call.
4. Verify the live site. Prefer the raw.githack URL (this is the one people can open today). Retry a few times if a CDN cache still shows the old `lastUpdated`.

```bash
curl -sI https://raw.githack.com/amanrubey/dehradun-traffic/main/index.html
curl -s https://raw.githack.com/amanrubey/dehradun-traffic/main/data.json | python3 -c "import sys,json; print(json.load(sys.stdin)['lastUpdated'])"
# Once GitHub Pages is enabled, also check:
curl -sI https://amanrubey.github.io/dehradun-traffic/
```

Expect HTTP 200 and the new `lastUpdated` value. The page fetches `data.json` with `cache: "no-store"`. Use `raw.githack.com` (short cache), not `rawcdn.githack.com` (long production cache).
