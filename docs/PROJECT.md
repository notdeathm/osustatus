# Project

A practical overview of how osustatus is built, why it is structured this way, and what tradeoffs the project makes.

## Goal

The project answers one question quickly and cleanly: how is this player actually doing right now? The product experience is designed around the next questions a player asks after seeing a profile:

1. Are they still improving?
2. What do they actually play?
3. How do they compare to someone else?

This is deliberately not a social product, not an account-based service, and not a beatmap marketplace. There is no database-backed user system and no paid backend. The app stays lightweight and static.

## Core architecture

The app is a static Next.js export hosted on GitHub Pages, with a Cloudflare Worker handling live osu! API access.

```text
src/
  app/            routes: /, /u, /vs, /rankings, 404
  components/     UI components, one concern per file
  lib/            client code, formatting, comparison logic, mod rules
shared/
  osu.mjs         raw osu! JSON normalization into app shapes
worker/
  src/index.ts    Cloudflare Worker proxy for live lookups
.github/workflows/
deploy.yml       static build and Pages publication
```

`next.config.ts` uses `output: "export"`, `basePath: "/osustatus"` by default, and `trailingSlash: true`. This lets the project deploy as a static site while still keeping links and asset paths stable under GitHub Pages.

## Why query parameters instead of pretty URLs

A static export cannot pre-render `/u/<any-name>` for an infinite set of possible player names. Instead, player pages live at `/u/?name=<player>`, and comparisons live at `/vs/?a=<player>&b=<player>`.

This is intentional and useful:

- The full route state is stored in the URL
- Search boxes can seed from the current URL without drift
- The path remains shareable and history-safe
- The app does not need server-side routing or a database backing the person lookup

The address is the single source of truth for the current view state.

## Data flow

`src/lib/osu.ts` is the single place that knows where data comes from. It resolves the public API base and routes all requests through the worker.

```ts
const API_BASE = (process.env.NEXT_PUBLIC_OSU_API_BASE || DEFAULT_API_BASE).replace(/\/+$/, "");
```

The app talks to the worker via a small set of endpoints:

- `GET {API_BASE}/api/player?u=&mode=&fresh=1`
- `GET {API_BASE}/api/rankings?mode=&country=&cursor=`
- `GET {API_BASE}/api/search?q=&mode=`

A deployment can point `NEXT_PUBLIC_OSU_API_BASE` at a self-hosted worker, but only the public address is exposed to the browser. The osu! secret remains in Cloudflare Worker secrets.

A generated sample profile in `src/lib/sample.ts` answers to the reserved name `~demo`, so the design can be inspected locally without making a live osu! request. The sample data is clearly labelled as sample data in the UI and is never a fallback for a real lookup.

Any stale JSON files under `public/api-data/` are archival only. The live app does not load those files.

## Error model and live-only behavior

The app normalizes errors into a small, human-readable set such as:

- `not_found`
- `restricted`
- `busy`
- `unavailable`
- `network`
- `bad_request`
- `unknown`

This allows a user to understand why a live lookup failed without exposing internal osu! API details or leaving the UI in a broken state.

## osu! API facts worth recording

The project depends on a few non-obvious osu! API constraints that are easy to miss:

| Fact | Why it matters |
| --- | --- |
| No CORS headers on osu! API responses | The browser cannot call osu! directly; a proxy is required |
| Client secret cannot live in a static site | It must stay in Cloudflare Worker secret storage |
| OAuth token is reused for roughly an hour | The worker keeps a token and refreshes it when needed |
| Score endpoints need `x-api-version: 20220705` | Without it, lazer plays can appear as zero-scored |
| A score is lazer when it has no `legacy_score_id` | `legacy_total_score` and `total_score` are not interchangeable |
| Profile accuracy is a percentage while score accuracy is a fraction | Normalization must happen once in `shared/osu.mjs` |
| Missing hit counts are omitted from API payloads | A missing value means zero, not unknown |
| Top plays are paginated at 100 per page | Two calls are needed for a 200-play window |
| Rank history is not in the API | The worker records its own observations over time |
| Covers are under `beatmapset.covers` | The correct card asset is `card@2x` |
| Flags use a specific osu! SVG path | Emoji flags render poorly on Windows |
| Country rankings return a cursor | Pagination is required for larger leaderboards |

## Rate budget and caching

osu! expects no more than about 60 requests a minute.

The worker uses a best-effort in-memory token bucket per isolate, then backs off and retries once after a `429` and `Retry-After` response. This is not a global quota across all isolates.

Caching strategy:

- Worker cache: 60 seconds in KV when available, otherwise in isolate memory
- UI freshness: the app shows a refresh hint once data is older than a minute
- Rank history: one daily observation plus additional same-day changes when detected, capped to roughly the latest year

## Analysis engine

The analysis layer in `src/lib/stats.ts` derives playstyle and trend information from the available top scores. It is intentionally based on the top 200 scores, not every play on the account.

### Improvement velocity

The project computes an activity ratio using recent top plays versus earlier history:

```text
recent90 = top plays in the last 90 days
usual90  = earlier top plays × 90 / (days of history before those 90)
ratio    = recent90 / usual90
```

The output ranges from "Inactive" to "Actively Improving" depending on pace and recency.

### Archetypes and traits

The app summarises players by mod-family popularity and a set of readable traits such as:

- HD Stacker
- HDDT Stacker
- Acc Machine
- Combo Master
- Marathon Runner
- Speed Demon
- Chill Player
- Grinder

These are written as clear player-facing descriptions, not opaque statistics.

## Decisions and tradeoffs

### Deliberately not done

- Mod-adjusted star rating and pp-if-FC via `rosu-pp-js`, because the static export and WASM asset path introduce risk
- Session-based social features or a backend user system
- Any database-backed requirement

### Already decided against

- Glowing row borders
- Multiple accent colours
- Inline browser title tooltips
- Forcing a second palette for light mode

## Accessibility and performance

- No external runtime fonts or analytics
- No osu! calls until the user opens a live view
- The theme is applied before paint to avoid a flash of light mode
- UI copy is written plainly and specific controls are keyboard accessible

## Roadmap

The app already has a solid base. The most likely future additions are:

1. Safer mod-adjusted PP calculations if static-export-friendly WASM constraints can be resolved
2. Open Graph metadata generation for shared links
3. Beatmap pages as another entity type
4. Offline installability or a stronger PWA layer

## Long-term risk areas

- osu! score model changes in lazer
- Rank history depth depending on data retention
- Sample data drift against `shared/osu.mjs`
- API version bumps that change response shapes
