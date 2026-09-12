# osu! local profiles

Repo: `github.com/jonathant09/osu-local-profiles`. That is the only name — do not reintroduce the old one.

## Before any new work

Read [`docs/roadmap.md`](docs/roadmap.md). Update its status column as you go.

Architecture notes: [`docs/architecture.md`](docs/architecture.md). Read relevant sections when needed.

## TS: strip-only, no runtime emit

`node src/main.ts` runs TS via Node type stripping. Types erased, never compiled.

- No `constructor(private db: Db)` — declare field + assign in body
- No `enum` (use `const` objects `as const`)
- No `namespace`, no decorators

`npm run check` = typecheck + full test suite. Use this.

Tests: `node --test "test/**/*.test.ts"` (quoted glob required).

## pp: official .NET helper only

`tools/PpCalculator/` wraps `ppy.osu.Game.Rulesets.*` NuGet. Driven via JSON-lines pipe from `src/calc/official.ts`.

- Hand the replay file, never a reconstructed ScoreInfo
- No fallback calc (`rosu-pp` removed). Helper down = store no pp, say so
- Do not trim it. `PublishTrimmed` breaks osu!'s own graph — see `docs/architecture.md`, roadmap 5.44
- Every pp carries osu! release version + breakdown. Parts must match the pp beside them
- `docs/reference-links.md` has links to osu-web, ppy/osu, API docs

## Design constraints

- No native modules in Node process. `node:sqlite` + pure-JS LZMA only
- `/api/profile` cached until DB changes (`total_changes()` stamp)
- No API polling. Local detection only. Works with no credentials, no network
- No scan-and-import on startup. Import past plays is explicit: pick cutoff, preview, confirm
- Ingestion serialized through promise queue (`src/tracker/index.ts`)

## UI checks

`npm run ui` drives `web/index.html` in headless Chrome via CDP, asserts computed style.

`[hidden] { display: none !important; }` is global. Keep it.

## Project layout

```
src/osr.ts              replay parser (legacy header + lazer extended block)
src/clients/            install detection, beatmap MD5 index, online.db
src/tracker/            fs.watch, settle-on-write, ingest + dedupe
src/calc/               pp (official only), level, grades, aggregation
src/calc/official.ts    JSON-lines client for .NET calc
src/calc/eligibility.ts single "this score counts" definition
src/settings.ts         per-profile settings
src/tracking-filter.ts  which plays tracked (at ingest, never after)
src/tracker/recompute.ts recalc scores in place from replays
src/scores.ts           pin, order, hide scores; View Details data
src/update/             GitHub release check, unpack, swap
scripts/apply-update.mjs detached swapper (ships in every package)
tools/PpCalculator/     .NET helper
web/index.html          Phase 1 UI
web/score.html          /scores/<id> page
web/js/                 page modules (share-copy, static-mode, bundle)
```
