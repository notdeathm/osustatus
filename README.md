# osustatus

A calm, fast viewer for public osu! player profiles, rankings, recent activity, and top plays.

Live site: <https://notdeathm.is-a.dev/osustatus/>

osustatus is a static site that helps people answer three questions quickly:

- Are they still improving?
- What do they actually play?
- How do they compare to someone else?

It is an independent viewer and is not affiliated with osu! or the official osu! site.

> osustatus was inspired in part by [osustats.app](https://osustats.app). Visit the original site and support its creator.

## Overview

The app is intentionally simple:

- A static Next.js export published on GitHub Pages
- A Cloudflare Worker that keeps the osu! client secret off the browser
- Live data only for real lookups; there is no hidden snapshot fallback for general use
- A special `~demo` profile for local UI exploration without a live osu! API request

The application is built to be installable, readable, and useful on a phone or desktop without needing accounts, database hosting, or anything more than a static frontend plus a small proxy worker.

## Architecture at a glance

```text
Browser / GitHub Pages
        │
        ▼
Next.js static export
        │
        ▼
Cloudflare Worker (osu! API proxy)
        │
        ▼
osu! API v2
```

The worker is the only place that can keep osu! client credentials. The browser sees only a public worker URL.

## Quick start

```bash
npm install
npm run dev
```

Then open:

- <http://localhost:3000/osustatus/>
- <http://localhost:3000/osustatus/u/?name=~demo>

The `~demo` route is a labelled sample profile used for design review and local UI checks without making a live osu! request.

## Validation commands

```bash
npm run typecheck
npm run build
npm run check
```

`npm run check` is the project standard validation step.

## Security and credentials

The important rule is simple:

> Never put an osu! client secret in the repository, in a frontend `NEXT_PUBLIC_*` variable, or in browser code.

The secret belongs only in the Cloudflare Worker secret store. For local Worker development, keep it in `worker/.dev.vars`, which is gitignored.

Public values that are safe to expose:

- `NEXT_PUBLIC_OSU_API_BASE` — the public URL for the Cloudflare Worker
- `NEXT_PUBLIC_BASE_PATH` — optional mount path for custom static hosts
- `NEXT_PUBLIC_SITE_URL` — optional canonical site URL used by the app metadata

See [docs/SETUP.md](./docs/SETUP.md) for the complete setup and deployment workflow.

## Project documentation

- [docs/PROJECT.md](./docs/PROJECT.md) — product goals, architecture, and how the app works
- [docs/DESIGN.md](./docs/DESIGN.md) — design system, accessibility, and visual decisions
- [docs/SETUP.md](./docs/SETUP.md) — local development, deployment, and environment variables
- [CONTRIBUTING.md](./CONTRIBUTING.md) — how to contribute
- [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) — community standards and expectations

## Project layout

```text
src/app/            Application routes and app-wide layout
src/components/      UI components grouped by concern
src/lib/            Data, formatting, analysis, and mod logic
shared/osu.mjs      Raw osu! JSON normalization for app data shapes
worker/             Cloudflare Worker API proxy
public/api-data/    Legacy archive; not used by the live app
docs/               Architecture, design, and setup docs
```

## Deployment

The project is designed as a static export. The default public deployment is GitHub Pages, with a Cloudflare Worker for live API access.

For the full deployment walkthrough, see [docs/SETUP.md](./docs/SETUP.md).

## Contributing

Contributions are welcome. Please read [CONTRIBUTING.md](./CONTRIBUTING.md) before opening a PR and follow [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md).

## License

osustatus is distributed under the GNU General Public License v3.0. See the [official license text](https://www.gnu.org/licenses/gpl-3.0.html).

## Attribution

Player data comes from the osu! API and belongs to osu! and its players. osustatus is a viewer, not a competitor, and respects osu!'s API terms.

## Ai Usage
GitHub Copilot assisted with research, planning, documentation, and parts of the implementation. Changes are reviewed and tested by me.
