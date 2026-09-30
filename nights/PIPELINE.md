# Nights: how the data gets refreshed

The site at `/nights/` is only a viewer. Everything it shows comes from one JSON file per city in `nights/data/`.
Today those files are filled by hand-checked research (the proof of concept). The next step replaces that with a scheduled job.

## Files

| File | What it is |
|---|---|
| `data/cities.json` | The dropdown: slug, name, country |
| `data/<slug>.json` | One city: currency, time zone, coverage note, sources, events |
| `sources.json` | Which listing sites work per city when read as plain HTML (tested 30 Sep 2026) |
| `index.html` | The page. Shows events from today to today + 13 days in the city's time zone |

## Event schema (one item in `events`)

```json
{
  "id": "maxcooper",             // unique within the city
  "date": "2026-10-03",          // local date the night starts
  "end": "2026-10-04",           // optional, multi-day events
  "time": "23:00",
  "title": "Max Cooper all night long, DVS1",
  "genre": "Techno",             // Techno, House, Melodic, Trance, Psytrance, Bass, Drum & Bass, EDM, Hip-hop / House, Retro, Day party, Festival, Club night, Live / Rock
  "score": 4.6,                  // 1–5 pick score
  "venue": "fabric", "area": "Farringdon", "addr": "fabric, 77a Charterhouse St",
  "lineup": "…", "desc": "One or two plain sentences.",
  "price": "From £24.21",        // local currency, or "Not listed publicly · shown at checkout"
  "link": "https://…",           // the event's own ticket/event page, never a venue homepage
  "artists": ["Max Cooper"],     // used for YouTube / Spotify buttons
  "vids": [{"id": "YouTubeID", "label": "Artist – Track"}]  // optional, plays on the page
}
```

## Planned Gemini job (not built yet)

1. A GitHub Action runs once a day (cron).
2. For each city it calls the Gemini API (free tier, a Flash model) with Google Search grounding switched on, plus the text of the `works` sources for that city from `sources.json`.
3. The prompt asks for events from today to today + 13 days, in exactly the schema above, with the event's own page as `link`.
4. The script validates the JSON (dates in window, link is https, no duplicate ids), keeps any hand-added `vids`, and commits `data/<slug>.json`.
5. GitHub Pages redeploys and the site updates. The page itself does not change.

Keep the API key as a GitHub Actions secret (`GEMINI_API_KEY`). Never put it in `index.html`: the site is public, and anyone could read and use the key.

Rate limits and quotas on the free tier change, so check the current Gemini API pricing page before scheduling 10 cities a day.
