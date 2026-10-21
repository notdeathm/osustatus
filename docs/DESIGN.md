# Design

A calm, quiet interface for a loud game.

The product borrows from the principle behind [osustats.app](https://osustats.app): one column, one card per idea, and no decoration that does not carry meaning. The visual language also follows the tone of [notdeathm.is-a.dev](https://notdeathm.is-a.dev): quiet surfaces, familiar typography, and restrained emphasis.

## Design principles

### 1. One accent, used deliberately

Pink `#ff75ae` and its stronger variant `#c52d6c` mark clickable, selected, or newly changed UI. Nothing else competes with that accent.

### 2. Colour should mean something

Only information-bearing colours are allowed to stand out: mod colours, accuracy tiers, rank movement, judgement counts, and chart states. This keeps the UI readable without becoming a rainbow.

### 3. Numbers should not jitter

All stats, scores, and percentages use tabular numerals so values do not shift visually when they update.

### 4. One frame everywhere

Cards share the same border, radius, and subtle shadow treatment. There are no floating panels, nested chrome, or decorative framing tricks.

### 5. No stray text

Ambiguous values are explained via the shared tooltip pattern rather than browser-native `title` hover text.

### 6. Grey means filtered, never disabled

When a time filter hides older plays, those rows remain on screen in greyscale so the user can see the cost of narrowing the window without losing context.

## Tokens

The palette is defined in `src/app/globals.css` via CSS custom properties, with a light-mode override under `[data-theme="light"]`. Tailwind v4 reads them through `@theme inline`, so the theme can flip without duplicating a second utility layer.

| Token | Dark | Light | Purpose |
| --- | --- | --- | --- |
| `--page` | `#13151b` | `#f1f2f4` | Page background |
| `--surface` | `#1a1d25` | `#ffffff` | Cards and cover overlays |
| `--surface-2` | `#222630` | `#f7f7f8` | Inputs and chips |
| `--surface-3` | `#2b303b` | `#eceef1` | Tooltips and nested sections |
| `--line` | `#30343e` | `#d9dce1` | Borders and dividers |
| `--line-strong` | `#444a57` | `#bfc4cd` | Emphasized separators |
| `--ink` | `#f1f0f6` | `#14161f` | Primary text |
| `--ink-soft` | `#c2c3cf` | `#3b3f4e` | Body copy |
| `--muted` | `#989ba8` | `#5c6172` | Secondary labels |
| `--faint` | `#7f8494` | `#6d7285` | Quiet detail text |
| `--accent` | `#ff75ae` | `#d93d7d` | Click targets and emphasis |
| `--accent-strong` | `#c52d6c` | `#c52d6c` | Filled buttons |
| `--accent-soft` | `#34212d` | `#f9e8ef` | Soft tint |
| `--green` / `--red` | `#62d2a0` / `#ff9292` | `#158f63` / `#c02626` | Improvement / drop |
| `--amber` | `#f0c674` | `#9a6b00` | Warning or slower pace |

Radii: 10px for cards, 6px for inner surfaces, and pill-shaped chips for toggles. Shadows are restrained and consistent.

## Type

The app uses the system UI stack only. There are no external font requests, no custom font download, and no runtime font dependency.

## Accuracy tiers

Accuracy is one of the few places where a value changes colour. The tiering is consistent and readable:

| Range | Colour |
| --- | --- |
| 100% | `#ff6b9d` |
| 99–100% | `#58d68d` |
| 97–99% | `#9ede6a` |
| 95–97% | `#e8d44d` |
| 90–95% | `#f0a15c` |
| below | `#ef6a6a` |

The light theme darkens the same tiers so small values still meet accessible contrast thresholds.

## Mod colours

Mod styling is defined once in `src/lib/mods.ts`. Each mod gets a background, border, and text treatment. Nightcore shares Double Time's family and Perfect shares Sudden Death's, because those are the same family semantics in osu!.

Badges over map covers use a fixed bright text colour so they remain readable over cover art.

## Player card

The player card is the most important interface element in the project. Because the osu! cover is arbitrary artwork, contrast cannot be left to chance.

- A continuous shadow rises from the bottom edge with pixel-based stops so the overlay behaves consistently across card heights.
- Two softened pools sit behind the bottom-left and bottom-right text blocks so names and ranks remain readable over busy covers.
- A muted darkening is used rather than a bright gradient, because the latter feels like a bug on top of cover art.
- On smaller screens, the banner sits behind the header and the shadow treatment becomes top-to-bottom instead of bottom-only.

## Score rows

Rows are laid out like osu!'s own profile list on large screens, but collapse into compact cards at narrower widths. The layout is intentionally disciplined so numbers remain easy to scan.

Key behaviors:

- The map cover fills the left 62% and gently fades to avoid a visible edge.
- Wide screens use one row with rank, title, mods, stats, and pp aligned.
- Under about 950px, the same item becomes a four-line card.
- The pink left-edge bar appears only while a filter is active.
- Filtered-out plays stay visible in greyscale with a darker overlay, but keep accuracy colours so they can still be scanned.

## Activity chart

The activity chart is intentionally simple: 18 months, one bar per month, with empty months omitted. The current month is grey, normal months use the accent, and the peak month takes the strongest emphasis.

The chart legend lists only the colours that are actually present, and the verdict badge shares the same calculation as the copy so they cannot contradict one another.

## Already decided against

These ideas were tested and then dropped so they do not keep resurfacing:

- Emoji badge walls
- Glowing borders on top-play rows
- A second accent colour
- Percentage-scaled card shadows
- Browser-native `title` hover text
- Sparklines inside list rows

## Accessibility

- Contrast is designed to meet AA on cards in both themes.
- All controls are real buttons, links, inputs, or selects.
- The chart bars are exposed with `role="img"` and an `aria-label`.
- The playstyle explainer dialog moves focus into it, closes on `Escape`, and closes when the backdrop is clicked.
- Tooltips are not the only way to get information; the same content is available in copy and dialogs.
- `prefers-reduced-motion` disables non-essential motion and transitions.
- The layout is designed to remain usable down to about 360px.
- The language is plain and direct: no unexplained abbreviations, no jargon without explanation.

## Final design goal

The product should feel quiet, intentional, and fast. It should let a visitor understand a player's recent performance without turning the page into a dashboard of noise. The interface should always stay readable, explain its own logic, and make the numbers feel trustworthy rather than decorative.
