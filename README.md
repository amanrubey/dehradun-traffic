# Dehradun traffic dashboard

Public, shareable dashboard of driving times on five Ballupur-area routes in Dehradun, India.

**Live site:** https://raw.githack.com/amanrubey/dehradun-traffic/main/index.html

The files in this repository are the source. The intended GitHub Pages address is https://amanrubey.github.io/dehradun-traffic/, but Pages has to be turned on once in the repo settings (Source: GitHub Actions) before that URL works. A deploy workflow is already in `.github/workflows/pages.yml`.

Times come from Google Maps checks (mornings around 9 AM IST and evenings around 6 PM IST, with afternoon and rink-trip checks added later). The page reads `data.json` and draws the charts in the browser. No times are estimated.

To add a new reading and republish, follow [UPDATE.md](UPDATE.md).
