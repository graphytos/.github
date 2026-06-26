# tv-show.schema.json

The schema for the JSON file the library vault writes for each **TV show** entry.
File lives at `<vault>/<tmdbId>.json`. The SQLite index (`<vault>/.index/library.sqlite`)
holds everything else (descriptions, runtime, genres, MPAA, episode metadata, full
credits, etc.) — this file is intentionally minimal.

The only structural difference from the film schema is the `seasonsWatch[]` array,
which carries per-season and per-episode watch state so individual seasons or
episodes can be marked watched independently.

## Fields

```jsonc
{
  "tmdbId": 1396,
  "imdbId": "tt0903747",
  "kind": "tv",

  "name": "Breaking Bad",
  "year": 2008,
  "poster": "/ggFHVNu6YYI5L9pCfOacjizRGt.jpg",
  "backdrop": "/tsRy63Mu5cu022Q9DZ54ubdkdAW.jpg",
  "cast": [
    { "tmdbPersonId": 17419, "name": "Bryan Cranston", "order": 0 },
    { "tmdbPersonId": 84497, "name": "Aaron Paul",     "order": 1 }
  ],

  "seasons": 5,
  "episodes": 62,
  "showStatus": "ended",
  "seasonsWatch": [
    {
      "seasonNumber": 1,
      "watched": true,
      "watchedAt": "2026-05-10",
      "episodes": [
        { "episodeNumber": 1, "watched": true, "watchedAt": "2026-05-10" },
        { "episodeNumber": 2, "watched": true, "watchedAt": "2026-05-10" }
      ]
    },
    {
      "seasonNumber": 2,
      "watched": false,
      "watchedAt": null,
      "episodes": [
        { "episodeNumber": 1, "watched": false, "watchedAt": null }
      ]
    }
  ],

  "letterboxd": "",
  "trakt":      "https://trakt.tv/shows/breaking-bad",
  "imdb":       "https://www.imdb.com/title/tt0903747/",

  "status":   "watched",
  "tags":     ["re-watch"],
  "dates":    ["2026-05-10"],
  "rating":   10,
  "notes":    "All-time favorite.",
  "favorite": true
}
```

## Field reference

### Shared with films

All fields from [`film.schema.md`](./film.schema.md) apply unchanged. The TV entry
also has:

| field            | type                | provenance | notes |
| ---------------- | ------------------- | ---------- | ----- |
| `seasons`        | `integer`           | AUTO       | Total number of seasons. From TMDB `number_of_seasons`. |
| `episodes`       | `integer`           | AUTO       | Total number of episodes across all seasons. From TMDB `number_of_episodes`. |
| `showStatus`     | `string \| null`    | AUTO       | One of `"returning"`, `"ended"`, `"canceled"`, `"in_production"`, or `null` if TMDB's status doesn't map. |
| `seasonsWatch`   | `SeasonWatch[]`     | MANUAL     | Per-season and per-episode watch state. See below. |

### TV-only mapping from TMDB `status`

- `"Returning Series"` → `"returning"`
- `"Ended"` → `"ended"`
- `"Canceled"` → `"canceled"`
- `"In Production"`, `"Planned"`, `"Pilot"` → `"in_production"`
- anything else → `null` (field not written)

## Nested types

### `SeasonWatch`

```ts
{
  seasonNumber: number;           // key — which season
  watched: boolean;               // MANUAL
  watchedAt: string | null;       // MANUAL, ISO date or null
  episodes: EpisodeWatch[];       // per-episode state for this season
}
```

### `EpisodeWatch`

```ts
{
  episodeNumber: number;          // key — which episode
  watched: boolean;               // MANUAL
  watchedAt: string | null;       // MANUAL, ISO date or null
}
```

## PATCH shape

PATCH bodies accept only the MANUAL fields. Season/episode watch state is
updated by addressing entries by `seasonNumber` (and nested `episodeNumber`):

```jsonc
{
  "seasonsWatch": [
    {
      "seasonNumber": 2,
      "watched": true,
      "watchedAt": "2026-06-20",
      "episodes": [
        { "episodeNumber": 1, "watched": true, "watchedAt": "2026-06-20" },
        { "episodeNumber": 2, "watched": true, "watchedAt": "2026-06-20" }
      ]
    }
  ]
}
```

## Watch-state rules

- "Mark season watched" sets `watched: true` on every episode in that season
  and stamps `watchedAt` on the season.
- "Mark season unwatched" clears every episode and the season itself.
- Per-episode state stays editable independently of the season rollup.
- The top-level `status` and `dates` fields are **independent** of `seasonsWatch[]`
  — they record "I watched the whole show" once, while `seasonsWatch[]` records
  granular progress.

## What is NOT in this file (lives in SQLite)

Kept out of the JSON so files stay small and grep-friendly. Joined at response
time by the detail endpoint:

- `description` — show overview
- `runtime` — typical episode runtime
- `genres`, `mpaa`
- Per-season AUTO metadata: season `name`, `airDate`, `episodeCount`, `overview`, poster
- Per-episode AUTO metadata: episode `name`, `airDate`, `overview`, `stillPath`, runtime
- Full cast + crew (rows in `credits`)
- Person records (rows in `people`)
- Raw `/tv/{id}` and `/tv/{id}/season/{n}/episode/{m}` response blobs (rows in `entries_full` and `tv_episodes`)

## Field provenance

- **AUTO** — populated from TMDB on create, refreshable via the admin heal flow. Never accepted in PATCH.
- **MANUAL** — user-editable. Safe to accept in PATCH.
- **SERVER** — written by the server only. Client must not set them.

(This schema has no SERVER fields — server-managed `createdAt` / `updatedAt` are intentionally omitted.)

## Example: minimal file on create

```jsonc
{
  "tmdbId": 1396,
  "imdbId": "tt0903747",
  "kind": "tv",
  "name": "Breaking Bad",
  "year": 2008,
  "poster": "/ggFHVNu6YYI5L9pCfOacjizRGt.jpg",
  "backdrop": "/tsRy63Mu5cu022Q9DZ54ubdkdAW.jpg",
  "cast": [
    { "tmdbPersonId": 17419, "name": "Bryan Cranston", "order": 0 }
  ],
  "seasons": 5,
  "episodes": 62,
  "showStatus": "ended",
  "seasonsWatch": [],
  "letterboxd": "",
  "trakt": "",
  "imdb": "",
  "status": "to-watch",
  "tags": [],
  "dates": [],
  "rating": 0,
  "notes": "",
  "favorite": null
}
```

=======================
<br/>
copyright 2026