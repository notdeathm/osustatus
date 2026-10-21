# Contributing to osustatus

Thanks for helping improve osustatus.

This project is intentionally small and opinionated: a calm static app with a live osu! API proxy, no sign-in flow, and no database-backed social features. Keep changes aligned with that scope.

Please read and follow the [Code of Conduct](./CODE_OF_CONDUCT.md) when participating in issues, discussions, pull requests, or project spaces.

## Before you start

Read the core documentation, especially:

- [README.md](./README.md)
- [docs/PROJECT.md](./docs/PROJECT.md)
- [docs/DESIGN.md](./docs/DESIGN.md)
- [docs/SETUP.md](./docs/SETUP.md)

If a change touches setup, deployment, security, or public-facing behavior, update the relevant docs in the same change.

## Contribution workflow

1. Open an issue for anything larger than a minor typo or quick fix.
2. Fork the repository and create a branch.
3. Make a focused change with the smallest possible scope.
4. Run the relevant validation commands.
5. Submit a PR with a clear summary of what changed and why.

## Local setup

```bash
npm install
npm run dev
```

Open:

- <http://localhost:3000/osustatus/>
- <http://localhost:3000/osustatus/u/?name=~demo>

The `~demo` profile is the safe UI test path. It is intentionally labelled as sample data and is not a real osu! user.

## Validation commands

```bash
npm run typecheck
npm run build
npm run check
```

`npm run check` is the standard verification step for the frontend.

## Worker setup

Player lookups need the Cloudflare Worker. For local Worker development:

```bash
cd worker
npm install
npm run dev
```

The Worker expects local secrets in `worker/.dev.vars`:

```text
OSU_CLIENT_ID=your-client-id
OSU_CLIENT_SECRET=your-client-secret
```

The browser app should only point at a public Worker URL such as:

```bash
NEXT_PUBLIC_OSU_API_BASE=http://localhost:8787 npm run dev
```

Do not place osu! credentials inside `NEXT_PUBLIC_*` variables. They must remain in the Worker secret store or in the gitignored local Worker file.

## Project conventions

- Keep components focused and compositional.
- Prefer existing design tokens in `src/app/globals.css`; do not introduce one-off colour values in components.
- Keep UI language plain and readable.
- Respect accessibility: keyboard support, visible focus states, and reduced-motion preferences.
- Match the existing structure: static export, worker proxy, minimal live-only behavior.

## Commands summary

| Command | Purpose |
| --- | --- |
| `npm run dev` | Start the app locally |
| `npm run typecheck` | Run TypeScript checks |
| `npm run build` | Create the static export |
| `npm run check` | Run typecheck and build |
| `npm run worker:dev` | Start the Worker locally |
| `npm run worker:deploy` | Deploy the Worker |

## License

By contributing, you agree that your work may be distributed under the GNU General Public License v3.0. Keep copyright and license notices intact.
