# film.schema.json

The schema for the JSON file the library vault writes for each **film** entry.
File lives at `<vault>/<tmdbId>.json`. The SQLite index (`<vault>/.index/library.sqlite`)
holds everything else (descriptions, runtime, genres, MPAA, full credits, etc.)
— this file is intentionally minimal.

## Fields

```jsonc
{
  "tmdbId": 550,
  "imdbId": "tt0114369",
  "kind": "movie",

  "name": "Fight Club",
  "year": 1999,
  "poster": "/pB8BM7pdSp6B6Ih7QZ4DrQ3PmJK.jpg",
  "backdrop": "/87hTDiay9Y3bcb11BaTzZbcMrt1.jpg",
  "cast": [
    { "tmdbPersonId": 819, "name": "Edward Norton", "order": 0 },
    { "tmdbPersonId": 287, "name": "Brad Pitt",    "order": 1 }
  ],

  "letterboxd": "https://letterboxd.com/film/fight-club/",
  "trakt":      "https://trakt.tv/movies/fight-club-1999",
  "imdb":       "https://www.imdb.com/title/tt0114369/",

  "status":   "watched",
  "tags":     ["re-watch", "with-friends"],
  "dates":    ["2026-06-20", "2026-07-04"],
  "rating":   9,
  "notes":    "Rewatched with Sam. Still hits.",
  "favorite": true
}
```

## Field reference

| field        | type                  | provenance | notes |
| ------------ | --------------------- | ---------- | ----- |
| `tmdbId`     | `integer`             | AUTO       | Primary identifier. Used as the filename (`<tmdbId>.json`). Never edited by the user. |
| `imdbId`     | `string \| null`      | AUTO       | IMDb ID (e.g. `tt0114369`). `null` when TMDB doesn't provide one. |
| `kind`       | `"movie"`             | AUTO       | Always `"movie"` for film entries. |
| `name`       | `string`              | AUTO       | Title. Auto-populated from TMDB, manually editable. |
| `year`       | `integer`             | AUTO       | Release year. |
| `poster`     | `string`              | AUTO       | TMDB `poster_path`. Binary cached at `<vault>/.index/images/<tmdbId>__poster.jpg` and served as base64. |
| `backdrop`   | `string`              | AUTO       | TMDB `backdrop_path`. Cached and served the same way as `poster`. |
| `cast`       | `CastEntry[]`         | AUTO       | Top-billed cast (up to 20), sorted by `order`. Full cast + crew live in SQLite. |
| `letterboxd` | `string`              | MANUAL     | Full URL to the title's Letterboxd page. |
| `trakt`      | `string`              | MANUAL     | Full URL to the title's Trakt page. |
| `imdb`       | `string`              | MANUAL     | Full URL to the title's IMDb page. |
| `status`     | `"to-watch" \| "watched"` | MANUAL  | Viewing status. |
| `tags`       | `string[]`            | MANUAL     | User-defined tags (e.g. `["re-watch", "cinema"]`). |
| `dates`      | `string[]` (ISO date) | MANUAL     | ISO `YYYY-MM-DD` appended every time the film is watched. |
| `rating`     | `integer` (0–10)      | MANUAL     | Personal rating. `0` means unset. |
| `notes`      | `string`              | MANUAL     | Free-form personal notes. |
| `favorite`   | `boolean`             | MANUAL     | `true` = favorited, absent/`null` = not favorited. |

## Nested types

### `CastEntry`

```ts
{
  tmdbPersonId: number;  // AUTO, TMDB person ID
  name: string;          // AUTO, display name
  order: number;         // AUTO, billing order (0 = top-billed)
}
```

## What is NOT in this file (lives in SQLite)

These are cached from TMDB into the SQLite index and joined at response time by the detail endpoint. Kept out of the JSON so files stay small and grep-friendly:

- `description` — plot overview / synopsis
- `runtime` — runtime in minutes
- `genres` — genre names
- `mpaa` — MPAA rating
- Full cast + crew (rows in the `credits` table)
- Person records (rows in the `people` table)
- Raw `/movie/{id}` response blob (row in `entries_full`)

## Field provenance

- **AUTO** — populated from TMDB on create, refreshable via the admin heal flow. Never accepted in PATCH.
- **MANUAL** — user-editable. Safe to accept in PATCH.
- **SERVER** — written by the server only. Client must not set them.

(This schema has no SERVER fields — server-managed `createdAt` / `updatedAt` are intentionally omitted.)

## Example: minimal file on create

```jsonc
{
  "tmdbId": 550,
  "imdbId": "tt0114369",
  "kind": "movie",
  "name": "Fight Club",
  "year": 1999,
  "poster": "/pB8BM7pdSp6B6Ih7QZ4DrQ3PmJK.jpg",
  "backdrop": "/87hTDiay9Y3bcb11BaTzZbcMrt1.jpg",
  "cast": [
    { "tmdbPersonId": 819, "name": "Edward Norton", "order": 0 },
    { "tmdbPersonId": 287, "name": "Brad Pitt",    "order": 1 }
  ],
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